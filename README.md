# HADO AirQo Android app — implementation package

This source project connects to the HADO WordPress website at `https://hado.org.ss/` through the companion **HADO AirQo Bridge** plugin.

## What's included

- Native Android (Kotlin + Jetpack Compose) dashboard.
- Grid discovery and grid selection (Kampala is selected by default when it appears in the grid list).
- Latest readings for PM2.5, PM10, PM1, temperature, relative humidity and CO.
- Additional pollutant trend selection: PM4, NO2, SO2, O3, CO2 and pressure where AirQo returns those fields.
- Date-range filters, 80-record pages, CSV export and five-minute live refresh.
- Link to AirQo's interactive map.
- Admin screen for Client ID, Access Token ID and Client Secret.
- WordPress plugin with REST endpoints, admin-only settings permissions and encrypted-at-rest credential storage.

## Important credential security

The Android app does **not** call AirQo directly and does not embed the AirQo secret/token. The admin enters them in the app, authenticated with a WordPress administrator Application Password. The WordPress plugin encrypts the values before saving them in the WordPress options table. The public app endpoints return readings only, never credentials.

The credentials visible in the originally uploaded PHP snippet should be considered exposed. Rotate the AirQo Client Secret and Access Token with AirQo before production deployment. Do not paste the old values into this project.

## Step 1 — install the WordPress bridge

1. In WordPress, open **Plugins → Add New Plugin → Upload Plugin**.
2. Zip the folder `wordpress-plugin/hado-airqo-bridge` (or upload the included `hado-airqo-bridge.zip`).
3. Activate **HADO AirQo Bridge**.
4. Confirm your site uses HTTPS and that the REST API is reachable at:
   `https://hado.org.ss/wp-json/hado-airqo/v1/health`
5. The expected health response contains `ok: true` and service `HADO AirQo Bridge`.

The existing dashboard shortcode and its old Code Snippets integration can remain in place while you test the Android bridge. The bridge uses the same AirQo endpoints and token query parameter as the uploaded integration, and sends the configured Client Secret header. It also sends Client ID as `X-Client-Id`; confirm with AirQo whether that header is required/accepted for your account.

## Step 2 — create an administrator Application Password

1. Sign in to WordPress as an administrator.
2. Open **Users → Profile → Application Passwords**.
3. Create one named `HADO AirQo Android Admin`.
4. Copy it once. This is a WordPress credential, not an AirQo credential.
5. Use the WordPress username and Application Password in the app's **ADMIN** screen. Do not use your normal WordPress login password.

The WordPress REST API must allow Application Password authentication over HTTPS. If your host, security plugin, or firewall blocks it, allow WordPress Application Password authentication for `/wp-json/hado-airqo/v1/settings`.

## Step 3 — configure AirQo credentials in the app

1. Open the app and tap **ADMIN**.
2. Enter the WordPress administrator username and Application Password.
3. Enter the newly rotated AirQo Client ID, Access Token ID and Client Secret.
4. Tap **Save securely to HADO website**.
5. Return to the dashboard. The app loads the available AirQo grids.

You can also configure the same credentials in **WordPress → Settings → HADO AirQo**. Blank fields in either settings form preserve the already stored value.

## Step 4 — build the Android app

Open this project folder in Android Studio (Giraffe or newer recommended), allow Gradle sync, then choose **Build → Build Bundle(s) / APK(s) → Build APK(s)**. The APK will be generated under:

`app/build/outputs/apk/debug/app-debug.apk`

For a production release, configure a private signing key in Android Studio and build a signed release APK/AAB. Do not publish a debug APK.

## API routes

- `GET /wp-json/hado-airqo/v1/health` — bridge health check.
- `GET /wp-json/hado-airqo/v1/grids` — available grids.
- `GET /wp-json/hado-airqo/v1/measurements?grid=GRID_ID&skip=0&limit=80` — readings.
- `GET /wp-json/hado-airqo/v1/settings` — admin-only credential status.
- `POST /wp-json/hado-airqo/v1/settings` — admin-only credential update.

The measurements endpoint is intentionally public because this is a public air-quality information dashboard. It validates requested grid IDs against the AirQo grid list, limits page size to 80 and caches responses briefly. If HADO wants a private staff-only app, add user authentication to the read endpoints before distribution.

## Notes and validation

- The uploaded dashboard code supports 80 records per page and five-minute refresh; these behaviors are retained.
- Historical retrieval uses `startTime` and `endTime`, matching the uploaded PHP implementation. That original code itself notes that long-range historical support should be confirmed with AirQo. The Android app cannot guarantee 20 years of historical records unless AirQo's endpoint and account return them.
- Some pollutants may show `—` when the selected AirQo grid/sensor does not report that field.
- This package has been checked for PHP syntax. It has not been built into an APK or tested against live AirQo credentials because the HADO host could not be reached from the build environment. Build and test from Android Studio after installing the plugin.


## App icon
The HADO AirQo launcher icon has been added in adaptive and legacy Android density formats. The launcher icon is configured in AndroidManifest.xml. A standalone 1024 x 1024 PNG is supplied separately as HADO-AirQo-App-Icon.png.
