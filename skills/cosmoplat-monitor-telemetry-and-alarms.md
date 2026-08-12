---
name: Monitor COSMOPlat device telemetry and work the alarm queue
description: Read live and historical telemetry for a COSMOPlat device, list the tenant's alarm records with severity and acknowledgement state, and record the handling of an alarm.
api: openapi/cosmoplat-iot-platform-openapi.yml
operations:
  - getTenantDevicesnew
  - getDeviceMetaByDeviceIdValuesTimeseries
  - getByEntityTypeByEntityIdAttributeProfilePage
  - postAlarmSettingPage
  - postAlarmByEntityTypeGetTenantAlarms
  - postAlarmSetAlarmDetailsByAlarmId
generated: '2026-08-11'
method: generated
source: https://www.cosmoplat.com/help/detail/304/1038
---

# Monitor COSMOPlat device telemetry and work the alarm queue

## Before you start

Read the "Before you start" section of
`skills/cosmoplat-onboard-product-and-device.md` — the same missing base URL and missing
documented authentication apply to every operation here.

## Steps

1. **Find the devices.** Call `getTenantDevicesnew` with `page=0`, `pageSize=30` and
   optionally `productManageId`, `name` (device identifier), `realName` (display name),
   `nodeType` or `active`. `active` is a boolean: `true` online, `false` offline. Note
   `lastActivityTime` (epoch millis) on each record — an `active` device with a stale
   `lastActivityTime` is worth flagging.

2. **Read telemetry.** Call `getDeviceMetaByDeviceIdValuesTimeseries` with the device ID in
   the path. The same operation serves both the latest values and a historical window;
   pass the documented query parameters for the window you want.

3. **Read attributes.** Call `getByEntityTypeByEntityIdAttributeProfilePage` with
   `entityType` and `entityId` in the path (use `DEVICE` and the device ID) plus `page` and
   `pageSize`.

4. **Understand what raises an alarm.** Call `postAlarmSettingPage` to page the alarm rules
   in force. Pagination for this operation lives in the **request body**, not the query
   string.

5. **List the alarm records.** Call `postAlarmByEntityTypeGetTenantAlarms` with
   `entityType` in the path. Each record carries `severity` (`CRITICAL`, `MAJOR`, `MINOR`,
   `WARNING`, `INDETERMINATE`), `status` (`ACTIVE_UNACK`, `ACTIVE_ACK`, `CLEARED_UNACK`,
   `CLEARED_ACK`), the `originator` reference object naming the device, and the
   `startTs` / `endTs` / `ackTs` / `clearTs` lifecycle timestamps.

6. **Record the handling.** Call `postAlarmSetAlarmDetailsByAlarmId` with the alarm ID in
   the path to write the handler details back onto the record. `handlerUserId`,
   `handlerUserNickname` and `handlerDetails` are null until this is done.

## Triage order that the data supports

Sort `ACTIVE_UNACK` first, then by `severity`, then by `startTs` ascending — the oldest
unacknowledged critical alarm is the one to work. `CLEARED_UNACK` records still need
acknowledging even though the condition has passed.

## On failure

See `errors/cosmoplat-problem-types.yml`. There is no published rate limit and no
`Retry-After` header, so back off on your own schedule rather than expecting the API to
tell you when to.
