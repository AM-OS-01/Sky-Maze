# Sky Maze

Sky Maze is an iPhone app for manually organizing booked travel into a trip timeline.

## Current flow

- Choose Flight, Hotel, Vacation Rental, Train, Rental Car, or Coach Bus.
- Enter the details and save. Flights, trains, and coaches support outbound, optional return, and connecting legs.
- Continue through the trip-building screen, review the cards, and save the trip to home.
- Swipe between standalone booking cards and their actions on home. Share and the ellipsis stay stationary above the pager; the menu targets the selected card. No page shows a neighboring-card preview.
- Booking cards use the shared frames 5–12 design on home, results, and Previous Trips: 356pt width, larger typography, adaptive height for long content, and aligned date/time rows. Home action tiles use 92pt notes and 90pt actions; the two-location rental shows pickup/dropoff, completion/weather, then Calendar.
- Home automatically focuses the relevant unfinished card on entry and app foreground. It never pulls the user away while browsing. See [home card focus](Docs/Home-Card-Focus.md).
- Use the permanent actions for confirmed locations, weather, and completion. The ellipsis contains Edit Details, Replace Trip, Previous Trips, and Privacy Policy, followed by a divider and a red Remove Trip action. The former two-option bottom sheet is removed; the separate previous-trip deletion sheet remains.
- Travel Note opens one editable, autosaving text note per booking (a connected journey shares one note). The tile changes to View Note once populated. Notes remain offline on the device with the trip, including in Previous Trips, where each card has a note link. Calendar and Share remain disabled.
- Notes use a native source-to-sheet zoom on iOS 18+; older systems and Reduce Motion use the standard sheet presentation. Done flushes pending text before closing; save failures keep the editor open with Retry. Typing stays local to the editor with debounced saves, also flushed when the app becomes inactive.

The active flow does not run OCR, document extraction, a parser, AI prompts, automatic corrections, or booking reconciliation.

## Intentionally retained components

- Screenshot upload UI: photo picker, image selection, thumbnails, and selection limits. This is a reusable component, not an active screen route. Its Continue action must be supplied by a caller; it does not start extraction.
- Scan storage/recovery: protected temporary files, atomic checkpoints, interruption cleanup, and cold-launch cleanup. Legacy stored text is decoded only for compatibility; no text-recognition engine remains.
- Provider recognition: airline, hotel, rail, coach, rental-company, and booking-source directories and resources.

Saved-trip storage, shared booking cards, manual-entry validation, airport lookup, and maps remain. The time-of-day plane animation appears only on the no-active-trip home; the former active-trip globe animation has been removed.

## Run

Open `SkyMaze.xcodeproj` in Xcode and run the `SkyMaze` scheme on an iPhone simulator or device.

Design previews require `SKY_MAZE_ENABLE_DESIGN_FIXTURES=1` and a supported `-SkyMazeFrame…` argument. They use in-memory trip storage. The removed upload-processing/failure routes are not preview destinations.

The current manual-entry debug check is `-SkyMazeParsedManualEntryReview`; the word “Parsed” is only a legacy launch-argument prefix, not an extraction dependency. Old OCR/parser/AI test suites and fixtures have been removed.

## Booking models and compatibility

The manual flow uses `TravelRecord`, typed `Travel…` booking models, and `TravelEntry`. `bookingGroupID` links outbound and return entries; it is not an email thread. Entries do not carry email senders, message IDs, source domains, or received dates.

Saved trips retain the existing SwiftData schema and JSON keys. `TravelRecord.title` still encodes under the original `subject` key. `LegacyTravelMetadata` preserves previously saved provenance/status fields when reading, editing, and saving older trips; it does not enable extraction or automatic booking updates.

`Website/index.html` is the local privacy-page source. Editing it does not publish changes to the hosted policy.

See [retained scan storage](Docs/Scan-Recovery.md) and [home animations](Docs/Home-Time-Of-Day.md).
