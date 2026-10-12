# AAP V4 host/plug-in discovery checklist

> **Informative guidance, not a normative interoperability specification.** This document covers discovery prerequisites only. It does not certify cross-version binary compatibility or successful real-time processing. It is a small contribution proposed under [issue #145](https://github.com/atsushieno/aap-core/issues/145).

AAP hosts and plug-ins are separately installed Android applications. The host must first *find* the plug-in service, then read its advertised metadata, connect to it, and negotiate any optional capabilities. Successfully finding a service does **not** imply that the service will instantiate, process audio, restore saved state, or render its UI successfully.

## Plug-in application

- Package the plug-in in an APK/AAB that contains a declared, exported `org.androidaudioplugin.AudioPluginService`.
- On the service's `<intent-filter>`, declare the exact action `org.androidaudioplugin.AudioPluginService.V4`.
- On the same service, declare the `org.androidaudioplugin.AudioPluginService.V4#Plugins` `<meta-data>` key, with `android:resource="@xml/aap_metadata"`.
- Provide `res/xml/aap_metadata.xml` and describe **each** plug-in in the package. A single Android service can advertise multiple plug-ins, so do not assume a one-to-one relationship between services and plug-in identities.
- Give each plug-in a stable `unique-id` if it is intended to remain identifiable across installed versions and project/preset restoration.
- Advertise optional features as AAP extensions in the metadata, rather than expecting a host to infer their presence.

A concrete multi-plug-in example is [`samples/aappluginsample/src/main/res/xml/aap_metadata.xml`](../../samples/aappluginsample/src/main/res/xml/aap_metadata.xml): it describes an instrument, an effect and an AAPXS example beneath one `<plugins>` root. Its Android service declaration is in [`samples/aappluginsample/src/main/AndroidManifest.xml`](../../samples/aappluginsample/src/main/AndroidManifest.xml).

## Host application

- For Android package visibility on Android 11+, declare the V4 service action in a `<queries><intent><action ... /></intent></queries>` element in the host manifest, as shown in [`samples/aaphostsample/src/main/AndroidManifest.xml`](../../samples/aaphostsample/src/main/AndroidManifest.xml).
- Query for services advertising `org.androidaudioplugin.AudioPluginService.V4`, for example with `AudioPluginHostHelper.queryAudioPluginServices()` (using Android's `PackageManager.queryIntentServices()`).
- Load the service's V4 `#Plugins` XML metadata to enumerate available plug-ins, **without assuming one plug-in per service**.
- Bind to the selected service using the supported AAP host APIs, and handle unavailable/disconnected services as errors rather than silently treating discovery as activation.
- Check declared extensions before invoking optional features. A host must not send an extension request that the plug-in does not declare.

## Boundaries

- **Protocol version:** V4 advertises a different service action from V3. Do not infer that a V3-only service is a compatible V4 plug-in. See [`docs/HACKING.md`](../HACKING.md) for the version history.
- **Extension support:** a recognized service and plug-in ID alone do not promise that optional parameters, state, MIDI, GUI, buses or other extensions are available. Read declarations and handle absence.
- **Operational correctness:** discovery does not establish host/plug-in interoperability across application versions, audio timing, thread safety, process lifetime, state/preset formats or native/Web UI protocols.
- **Scope:** this checklist does not prescribe a new AAP API, XML schema, state machine, or conformance profile. Those remain outside this limited documentation proposal.

## How to check your implementation

1. Compare the host and plug-in manifests with the two sample manifests linked above.
2. Verify that a compatible plug-in package is visible in service discovery on an Android 11+ test environment.
3. Verify that its metadata is readable and each declared plug-in has a distinct identity.
4. Test connection, instantiation and processing separately; do not report them as passing on the basis of manifest inspection.
5. Test negative cases: an absent package, an unadvertised optional extension and an incompatible service version.
6. When doing runtime validation, use the repository's [host sample validator](../../samples/aaphostsample/README.md) and record which checks were actually executed.

For detailed APIs and extension declaration rules, see [`docs/DEVELOPERS.md`](../DEVELOPERS.md) and [`docs/EXTENSIONS.md`](../EXTENSIONS.md).
