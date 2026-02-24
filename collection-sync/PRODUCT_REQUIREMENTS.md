# Product Requirements Document (PRD): Apple Photos to Google Photos Sync Tool

## 1. Overview

This document outlines the product requirements for a tool designed to synchronize shared photo collections from Apple Photos to Google Photos. The primary goal is to enable users who participate in shared Apple Photos albums to consolidate these memories within their Google Photos library, which they use as their primary photo management platform.

The tool will be designed with data safety and user control as core principles.

## 2. Goals

*   Provide a reliable, user-initiated method to transfer photos from a shared Apple Photos collection into a Google Photos album.
*   Ensure the integrity of user's photo libraries by preventing data loss and avoiding the creation of duplicate images.
*   Preserve key photo metadata, specifically timestamps and location, to maintain the chronological and contextual accuracy of the user's memories.

## 3. Non-Goals

*   **Continuous Sync:** The tool will not run as a background service for continuous, automatic synchronization. All sync operations are manually triggered by the user.
*   **Two-Way Sync:** This is a one-way sync tool from Apple Photos to Google Photos only. Changes in Google Photos will not be synced back to Apple Photos.
*   **Comprehensive Metadata Sync:** The initial scope is limited to syncing photo timestamps and location data. Other metadata like comments, likes, and descriptions are not included.
*   **Destructive Operations:** The tool will not delete photos from the destination (Google Photos) album, even if they are removed from the source (Apple Photos) collection.
*   **GUI:** The initial version of this tool will not have a graphical user interface.

## 4. Target Audience

The primary user is someone who uses an iPhone and participates in shared Apple Photos albums with friends and family, but uses Google Photos as their central, long-term photo library and primary way of consuming photos.

## 5. User Requirements

| ID | Requirement | Description |
| :--- | :--- | :--- |
| **UR-1** | **Safety First** | All operations that modify data (uploading, creating albums) must be executed only after a "dry run". The tool must describe the specific actions it will take and require explicit user confirmation before proceeding. |
| **UR-2** | **Complete Collection Sync**| The tool must be able to access and sync all photos from a specified shared Apple Photos collection, regardless of which participant originally uploaded them. |
| **UR-3** | **Smart Inclusion of Existing Photos** | The tool must identify photos that already exist in the user's Google Photos library. If such a photo is part of the source Apple Photos collection, it should be added to the destination Google Photos album without being re-uploaded. This ensures all relevant photos from the Apple collection appear in the Google Photos album while preventing duplicate uploads to the user's main Google Photos library. |
| **UR-4** | **One-to-One Sync** | The tool's operation is focused on syncing one source Apple Photos collection to one destination Google Photos album at a time. |
| **UR-5** | **Destination Choice** | Users must be able to choose whether to sync their photos into a new Google Photos album (to be created by the tool) or into a pre-existing album. |
| **UR-6** | **Manual Trigger** | The sync process must be a discrete action, manually initiated by the user. This allows for both initial syncs and subsequent "top-up" syncs to fetch new additions. |
| **UR-7** | **Metadata Preservation** | The original timestamp (creation date/time) and location data (GPS coordinates) of each photo must be preserved when uploaded to Google Photos. |
| **UR-8** | **Application Type** | The initial version of the tool will be a Command Line Interface (CLI), prioritizing simplicity and automation. |
| **UR-9** | **(Optional) Identity Sharing** | As a non-essential "nice-to-have" feature, the tool could provide a mechanism for the user to share the Google Photos album with the same people from the Apple collection, likely via a manual mapping of identities in a configuration file. |


## 6. User Journeys

### Journey 1: First-Time Sync to a New Album

1.  **Initiation:** The user opens their terminal and executes the sync command.
2.  **Authentication:** The tool prompts the user to authenticate with their Apple and Google accounts (likely via an OAuth flow that opens a web browser).
3.  **Source Selection:** The tool prompts the user to specify the source Apple Photos collection (e.g., by displaying a numbered list of available shared collections to choose from).
4.  **Destination Specification:** The tool asks for the name of the new Google Photos album to be created.
5.  **Dry Run & Confirmation:** The tool performs an analysis and presents a summary:
    > "Found 150 photos in Apple collection 'Trip to Italy'.
- 125 photos are new to your Google Photos library and will be uploaded.
- 25 photos already exist in your Google Photos library and will be added to the new album.

A new Google Photos album named 'Trip to Italy' will be created."
6.  **User Consent:** The tool asks for explicit confirmation: "Proceed with sync? (y/N)".
7.  **Execution:** The user types 'y' and presses Enter. The tool creates the new album, uploads the 125 photos (ideally with a progress indicator), and adds them to the album.
8.  **Completion:** The tool displays a success message: "Sync complete. 125 photos added to new album 'Trip to Italy'."

### Journey 2: Subsequent Sync to an Existing Album

1.  **Context:** A week later, more photos have been added to the Apple collection. The user wants to sync the new additions.
2.  **Initiation:** The user runs the same sync command.
3.  **Authentication:** The tool uses cached credentials or prompts for re-authentication if sessions have expired.
4.  **Source/Destination Selection:** The user specifies the same source Apple Photos collection and the now-existing Google Photos album 'Trip to Italy'.
5.  **Dry Run & Confirmation:** The tool performs its analysis:
    > "Found 165 photos in Apple collection 'Trip to Italy'.
- 15 photos are new to your Google Photos library and will be uploaded to the existing album.
- 150 photos already exist in your Google Photos library. All 150 will be confirmed to be in the album; any not already present will be added."
6.  **User Consent:** The tool asks for confirmation: "Proceed with sync? (y/N)".
7.  **Execution:** The user confirms. The tool uploads only the 15 new photos to the existing Google Photos album.
8.  **Completion:** The tool displays a success message: "Sync complete. 15 new photos added to album 'Trip to Italy'."
