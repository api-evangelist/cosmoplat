---
name: Dispatch a command to a COSMOPlat device
description: Send a one-way or two-way RPC command to a device connected to the COSMOPlat IoT development platform, with the safety checks the API's own contract does not provide.
api: openapi/cosmoplat-iot-platform-openapi.yml
operations:
  - getDeviceInfoByDeviceId
  - postRpcOnewayByDeviceId
  - postRpcTwowayByDeviceId
generated: '2026-08-11'
method: generated
source: https://www.cosmoplat.com/help/detail/304/1038
---

# Dispatch a command to a COSMOPlat device

## Read this first

These two operations dispatch commands to **physical industrial equipment**. The published
contract gives you no idempotency key, no request identifier, no documented error catalogue
for the dispatch path and no rate limit. Everything protective here has to be done by the
caller.

## Steps

1. **Confirm the device is online.** Call `getDeviceInfoByDeviceId` and check `active` is
   `true` and `lastActivityTime` is recent. Dispatching to an offline device has no
   documented behaviour.

2. **Choose the RPC mode deliberately.**
   - `postRpcOnewayByDeviceId` — fire and forget. No result comes back. Use only for
     commands that are safe to lose.
   - `postRpcTwowayByDeviceId` — request/response. Use whenever you need to know the
     command landed.

3. **Send the command** with the device ID in the path and the documented body fields.

4. **Handle a timeout as UNKNOWN, not as failure.** With no idempotency key, a re-sent
   command may execute twice on the equipment. On a timeout, verify the device's resulting
   state through telemetry
   (`getDeviceMetaByDeviceIdValuesTimeseries`) before deciding whether to re-send.

## Guardrails to apply in an agent

- Require an explicit human confirmation before either RPC operation. The rubric class for
  both is `acting` / `physical`; see the `x-agentic-access` annotations in
  `overlays/cosmoplat-iot-platform-overlay.yaml`.
- Keep your own dispatch log keyed by device ID and command, because the API will not
  de-duplicate for you.
- Prefer `twoway` over `oneway` unless the command is genuinely idempotent at the device.
