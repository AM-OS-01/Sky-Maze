# Sky Maze

Sky Maze is an iPhone app for manually organizing booked travel into a trip timeline.

## Current flow

- Choose Flight, Hotel, Vacation Rental, Train, Rental Car, or Coach Bus.
- Enter the details and save. Flights, trains, and coaches support outbound, optional return, and connecting legs.
- Continue through the trip-building screen, review the cards, and save the trip to home.
- Swipe between standalone booking cards and their actions on home. The upper-right toolbar scrolls with its card; the menu targets that card. No page shows a neighboring-card preview.
- Booking cards use the shared frames 5–12 design on home, results, and Previous Trips: 356pt width, larger typography, adaptive height for long content, and aligned date/time rows. Home action tiles use 92pt notes and 90pt actions; the two-location rental shows pickup/dropoff, completion/weather, then Calendar.
- Home automatically focuses the relevant unfinished card on entry and app foreground. It never pulls the user away while browsing. See [home card focus](Docs/Home-Card-Focus.md).
- Use the permanent actions for confirmed locations, weather, and completion. The ellipsis contains Edit Details, Add to Calendar, Add to Trip, and Previous Trips, followed by a divider and a red Remove Trip action. Add to Trip appends new events without replacing existing travel details. Privacy Policy is in the information menu. The separate previous-trip deletion confirmation remains.
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

## Privacy policy publishing

`Website/privacy.html` is the canonical privacy-policy source, last updated October 12, 2026. `index.html` redirects to it, and the support page links to it. The app's information menu and subscription screen both use `https://am-os-01.github.io/Sky-Maze/privacy.html`.

Publish `privacy.html`, `index.html`, `support.html`, and `styles.css` together to the GitHub Pages site root. Editing these local files does not publish changes to the hosted policy. Verify the hosted `/Sky-Maze/privacy.html` before release. The app URL does not need to change.

The policy covers scanning through Railway/OpenAI, local files and recovery, traveler sharing, Apple services, StoreKit, and privacy requests. The earlier manual-only implementation notes above are historical and are not a privacy description of the current scanning/subscription build.

### Privacy implementation audit — October 12, 2026

- `ScreenshotUploadPreparation` re-encodes images; `PDFImportPreparation` renders pages. The scanning wire format contains images, not original PDFs or file names. Visible personal information is not redacted.
- `Backend/app/extraction.py` uses `store: false`; this is not zero provider retention. The policy uses [OpenAI's published data controls](https://developers.openai.com/api/docs/guides/your-data), not an assumption about the production account's settings.
- `Backend/app/device_auth.py` persists installation registrations without automatic expiry. Challenge cleanup and `ledger.py` daily cleanup are request-driven, not scheduled deletion guarantees. The Docker command disables Uvicorn access logging, but hosting logs and backups still need operator review.
- `TravelerTripFile` shares selected records/travelers and optional notes/complete confirmations as independent copies. `TripSharePDF` remains a separate export path. No live cloud collaboration or App Store invite-link behavior is claimed.
- `SubscriptionStore` reads verified StoreKit entitlement/offer information on device. It does not send transaction data to the scanning backend. Local StoreKit testing is not a live subscription launch.
- `SkyMaze/PrivacyInfo.xcprivacy` now declares photos, other user content, installation identifiers, scan interactions, and diagnostics for app functionality; linked is conservatively true because content may identify travelers and request records link to installations. Tracking remains false. The existing UserDefaults required-reason declaration is preserved.
- Payment-card information is handled by Apple, not received by this code. On-device trip data and on-device StoreKit state alone are not declared as off-device collection. Reassess Purchase History if server-side purchase verification or transaction storage is added. Review any specifically extracted personal fields, support intake, and hosting metadata before submitting final privacy labels.

### Remaining release checks

1. Publish these website files and verify the exact URL from the app on a device. No hosting deployment or App Store Connect changes are performed by a local edit.
2. Confirm actual Railway log/backup retention, OpenAI data-sharing/retention settings, processing regions, and the operator's privacy-request handling. Do not claim zero retention or no training without confirming account settings.
3. Review the final policy for the intended launch jurisdictions and operator identity. This implementation-based draft is not a legal compliance certification.
4. Review the pre-scan disclosure and explicit permission for sharing personal data with the AI provider before public release; updating a policy does not implement a consent flow.
5. Update App Store Connect privacy answers separately. The bundled manifest does not publish privacy labels. Use [Apple's collection and linkage guidance](https://developer.apple.com/app-store/app-privacy-details/) and [manifest data types](https://developer.apple.com/documentation/bundleresources/app-privacy-configuration/nsprivacycollecteddatatypes/nsprivacycollecteddatatype).

See [retained scan storage](Docs/Scan-Recovery.md) and [home animations](Docs/Home-Time-Of-Day.md).
