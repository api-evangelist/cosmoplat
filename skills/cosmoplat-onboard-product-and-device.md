---
name: Onboard a COSMOPlat product and its first device
description: Create a product definition on the COSMOPlat IoT development platform, define its thing-model telemetry points, register a device against it, and read back the device's access token so the device can publish telemetry over MQTT.
api: openapi/cosmoplat-iot-platform-openapi.yml
operations:
  - getRuleChains
  - postTbProductManage
  - postMergerTelemetryProfile
  - getPageByTenantIdAndProductId
  - postDevice
  - getDeviceInfoByDeviceId
generated: '2026-08-11'
method: generated
source: https://www.cosmoplat.com/help/detail/304/1038
---

# Onboard a COSMOPlat product and its first device

## Before you start

- **You cannot call this API from the documentation alone.** COSMOPlat publishes no base
  URL and no authentication scheme for these operations. Get the host and the credential
  from the 物联网平台管理员 (IoT platform administrator) who provisions your tenant, or from the
  卡奥斯天云 (CUBA) tenant you sign in with. See `authentication/cosmoplat-authentication.yml`.
- Every request and response body is `application/json`.
- **There is no idempotency key on this API.** If a create call times out, do not blind
  retry — list first and check whether the record already exists.

## Steps

1. **Pick a rule chain.** Call `getRuleChains` with `page=0` and `pageSize=30`. A product
   requires `defaultRuleChainId`, so choose one from the response before creating
   anything. Reference it as `{"id": "<ruleChainId>", "entityType": "RULE_CHAIN"}`.

2. **Create the product.** Call `postTbProductManage` with `name`, `nodeType`
   (`DIRECT_DEVICE`, `GATEWAY_DEVICE` or `GATEWAY_CHILD_DEVICE`), `transportType`
   (`DEFAULT`, `MQTT`, `LWM2M`, `SNMP` or `CoAP`) and the `defaultRuleChainId` object from
   step 1. All four are required. Keep the returned `id.id` — it is the product ID.

3. **Define the thing-model telemetry points.** Call `postMergerTelemetryProfile` to save
   or modify the measurement points (测点) on the product. The `name` of each point is the
   key a device must use in its telemetry payload; a mismatch means the value is silently
   not delivered to subscribers. Confirm with `getPageByTenantIdAndProductId`, passing
   `tbProductManageId` = the product ID plus `page` and `pageSize`.

4. **Create the device.** Call `postDevice` with the device fields and the
   `productManageId` reference object `{"id": "<productId>", "entityType":
   "TB_PRODUCT_MANAGE"}`. Keep the returned `id.id` — it is the device ID.

5. **Read the device credential.** Call `getDeviceInfoByDeviceId` with the device ID. The
   response carries `credentialsType` (`ACCESS_TOKEN`, `X509_CERTIFICATE` or `MQTT_BASIC`)
   and `deviceCredentialsId`, which is the device's access token. Treat it as a secret —
   note that the same value is also returned in bulk by `getTenantDevicesnew`.

6. **Publish telemetry.** The device connects to `iot.cosmoplat.com:1883` and publishes to
   `v1/devices/me/telemetry` with a flat JSON object whose keys are the telemetry point
   identifiers from step 3. See `asyncapi/cosmoplat-iot-telemetry-asyncapi.yml`.

## Conventions to respect

- **Pagination** on GET list operations is `page` (zero-based) and `pageSize` in the query
  string; responses carry `totalPages`, `totalElements` and `hasNext`.
- **Every foreign key is an object**, `{entityType, id}` — never a bare id.
- **Timestamps** on entities are epoch milliseconds.

## On failure

A failure returns `{"status", "message", "errorCode", "timestamp"}` where `status` mirrors
the HTTP status and `message` is in Chinese. No error-code registry is published, so do not
branch on `errorCode` values you have not seen. If the host you were given is the gateway
at `openapi.cosmoplat.com`, note that it answers failures with **HTTP 200** and a different
envelope, `{"errCode", "errMsg"}` — check the body, not just the status. See
`errors/cosmoplat-problem-types.yml`.
