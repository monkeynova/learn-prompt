# Technical Design: Apple Photos to Google Photos Sync Tool

This document outlines the proposed technical architecture and implementation plan for the Apple Photos to Google Photos Sync Tool, based on the requirements in the [PRD](./PRODUCT_REQUIREMENTS.md).

## 1. Core Architectural Decisions

This section will be populated as key technical questions are investigated and answered.

### 1.1. Target Platform

**Decision:** Based on the need to access Apple Photos data, the initial version of this tool will be developed for and supported on **macOS only**.

**Justification:** Accessing Apple Photos libraries is most reliably achieved on the platform where the Photos app itself runs. Attempting to support Windows or Linux would introduce significant complexity and likely be impossible without undocumented and unstable iCloud APIs.

### 1.2. Language/Runtime

**Decision:** TBD. Likely candidates are Python or Go, based on library availability for interacting with the required APIs and local databases.

## 2. Key Technical Investigations

This section details the investigation into the core technical challenges identified during the initial PRD review.

### 2.1. Apple Photos Data Access

**Challenge:** How to reliably access photos and their metadata (timestamp, location) from a user's library, including shared albums.

**Conclusion: Use the `osxphotos` Python library.**

**Investigation Summary:**
*   **Official APIs:** Direct access via official Apple APIs like `PhotoKit` is overly restrictive and does not provide a reliable way to read photos from shared albums.
*   **Local Database Inspection:** The `osxphotos` library provides robust, read-only access to the local `Photos.photoslibrary` database on macOS. Crucially, it can list shared albums and retrieve the photos within them on supported macOS versions.

**Chosen Approach:**
The tool will be a Python CLI that uses the `osxphotos` library to perform the following:
1.  Locate the user's Photos library on disk.
2.  Request the necessary permissions from the user via the standard macOS privacy prompt (TCC).
3.  Use `osxphotos` to list all albums, including shared albums (`albums_shared`).
4.  For a selected album, retrieve all `Photo` objects.
5.  For each `Photo` object, extract the required metadata (timestamp, location) and export the original image file for processing.

**Constraints & Risks:**
*   **Platform:** The tool will be **macOS-only**.
*   **Python Version:** Requires Python 3.10+.
*   **macOS Version:** `osxphotos` supports macOS 10.15 (Catalina) and newer, but has known issues with very recent/future versions ("macOS Tahoe"). The tool's documentation must clearly state the supported OS versions. This is a manageable risk.

### 2.2. Google Photos API Integration

**Challenge:** How to upload photos, create/manage albums, and identify existing photos to prevent duplicates (UR-3).

**Conclusion: Use a local cache for duplicate detection; metadata preservation is only partially supported.**

**Investigation Summary & Chosen Approach:**

1.  **Duplicate Detection (UR-3):**
    *   **Finding:** The Google Photos API does **not** support searching for photos by a content hash.
    *   **Approach:** The tool will implement a local caching mechanism. It will maintain a simple database (e.g., a JSON or SQLite file) that stores a mapping of a photo's content hash (SHA-256) to the `mediaItem` ID returned by the Google API upon successful upload.
    *   **Workflow:** Before uploading a file, the tool will first check this local cache. If the hash exists, the upload is skipped, and the cached `mediaItem` ID is used to add the photo to the target album. This prevents re-uploads *by our tool* and satisfies the core need of UR-3.
    *   **Limitation:** This method cannot detect photos that were uploaded manually or by other applications. This is a necessary trade-off due to API limitations.

2.  **Metadata Preservation (UR-7):**
    *   **Timestamp:**
        *   **Finding:** The API correctly interprets the `DateTimeOriginal` EXIF tag embedded in the image file.
        *   **Approach:** The tool will export the original, unmodified image file using `osxphotos` and upload it directly. This will preserve the original creation timestamp. **This requirement is achievable.**
    *   **Location:**
        *   **Finding:** The Google Photos API **does not** support programmatically adding or modifying location (GPS) data due to platform privacy restrictions.
        *   **Approach:** Location data cannot be synced. **This requirement is not achievable.** The PRD must be updated to reflect this limitation.

3.  **Album Management (UR-5):**
    *   **Finding:** The API provides all necessary functionality.
    *   **Approach:** The tool will use `albums.list` to allow users to select an existing album, `albums.create` to make a new one, and `albums.batchAddMediaItems` to add photos to the selected album.

**Language/Runtime Decision:**
*   Based on the `osxphotos` library, the tool will be written in **Python**.

## 3. Proposed Workflow (High-Level)

This will be refined once the investigations are complete.

1.  **Initialization:** User runs the CLI tool.
2.  **Authentication:**
    *   **Google Photos:** Initiate OAuth 2.0 flow, storing credentials securely in the user's keychain for future runs.
    *   **Apple Photos:** Access will likely require user permission (TCC) to read the Photos library data.
3.  **Source Selection:** Use the chosen Apple Photos access method to list all available albums, including shared albums, for the user to select.
4.  **Destination Selection:** Use the Google Photos API to list all available albums, and prompt the user to choose one or provide a name for a new one.
5.  **Analysis (Dry Run):**
    a. Fetch all photo identifiers/metadata from the source Apple album.
    b. For each photo, generate a content hash (SHA-256).
    c. Use the Google Photos API to check which hashes already exist in the user's library.
    d. Compare the list of photos in the source Apple album against the contents of the target Google album.
    e. Generate and display the execution plan to the user.
6.  **Confirmation:** Await explicit user approval.
7.  **Execution:**
    a. For new photos, upload them to Google Photos with their original metadata.
    b. For all photos in the source collection, ensure they are present in the target Google Photos album, adding any that are missing.
8.  **Completion:** Report success to the user.
