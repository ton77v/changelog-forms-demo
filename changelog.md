# Changelog

<!-- AUTO-GENERATED from changelog/*.yaml — do not edit directly -->

### (2026-05-11) What's new in **ROR 1.69.1**
* **🚨Security Fix** (KBN) Fixed vulnerability [CVE-2026-2950](https://nvd.nist.gov/vuln/detail/CVE-2026-2950)
* **🚀New** (KBN) 9.4.0 9.3.4, 9.3.3, 9.2.8, 8.19.15, 8.19.14 support
* **🚀New** (ES) 9.3.4, 9.3.3, 9.2.8, 8.19.15, 8.19.14 support
* **🚀New** (ECK) 3.4.0 support
* **🐞Fix** (KBN) Fixed `jsonwebtoken-ancient` being stripped from Kibana builds earlier than 7.11.0

### (2026-04-02) What's new in **ROR 1.69.0**
* **🚨Security Fix** (KBN) [CVE-2026-24001](https://nvd.nist.gov/vuln/detail/CVE-2026-24001), [CVE-2025-69873](https://nvd.nist.gov/vuln/detail/CVE-2025-69873), [CVE-2026-2391](https://nvd.nist.gov/vuln/detail/CVE-2026-2391), [CVE-2026-25639](https://nvd.nist.gov/vuln/detail/CVE-2026-25639), [CVE-2026-27904](https://nvd.nist.gov/vuln/detail/CVE-2026-27904), [CVE-2026-3449](https://nvd.nist.gov/vuln/detail/CVE-2026-3449), [CVE-2025-15599](https://nvd.nist.gov/vuln/detail/CVE-2025-15599), [CVE-2026-33750](https://nvd.nist.gov/vuln/detail/CVE-2026-33750), [CVE-2026-4867](https://nvd.nist.gov/vuln/detail/CVE-2026-4867), [CVE-2026-34601](https://www.tenable.com/cve/CVE-2026-34601), [CVE-2022-31129](https://nvd.nist.gov/vuln/detail/cve-2022-31129)
* **🚀New** (KBN/ES) [Added Fleet support via native API key and service account token authentication (ES 7.14+)](https://docs.readonlyrest.com/elasticsearch/fleet)
* **🚀New** (KBN/ES) The ReadonlyREST Audit Dashboard available in the Kibana plugin now supports audit events written to data streams
* **🚀New** (KBN/ES) The ReadonlyREST Audit Dashboard provided by the Kibana plugin can now be used with the ECS (Elastic Common Schema) audit index
* **🚀New** (KBN) [Added support for opening different tenancies in separate tabs](https://forum.readonlyrest.com/t/multi-tenancy-and-link-sharing/1978/3)
* **🚀New** (KBN) [Added support for sharing links to Kibana visualizations for the selected tenancy](https://forum.readonlyrest.com/t/multi-tenancy-and-link-sharing/1978/3)
* **🚀New** (KBN) Added support for rolling upgrades when upgrading the ROR Elasticsearch plugin and ROR Kibana plugin in a cluster
* **🧐Enhancement** (KBN) Removed the need for manual username input in the impersonation mechanism
* **🧐Enhancement** (KBN) Fixed an error in Kibana caused by empty data streams in Kibana 8.18.0+
* **🧐Enhancement** (KBN) Added a fallback for an empty `indices` field in the Audit Dashboard
* **🧐Enhancement** (KBN) [Updated custom metadata examples to use the new method. `getIdentitySession` and `getAuthorizationHeaders` are now deprecated in favor of `getUserRequestIdentity`, `getIdentitySessionHeaders`, and `getWhitelistedHeaders`](https://docs.readonlyrest.com/develop/examples/custom-middleware)
* **🧐Enhancement** (ES) [`token_authentication` rule extended with `api_key` and `service_token` types](https://docs.readonlyrest.com/elasticsearch#token_authentication)
* **🧐Enhancement** (ES) [Audit log entries and ACL history now include a human-readable reason when a request is denied, making access-control troubleshooting significantly easier](https://forum.readonlyrest.com/t/distinguish-between-wrong-credentials-and-missing-permissions/2914)
* **🧐Enhancement** (ES) Added the new `matched_block_names` field to audit entries created by audit log serializers other than ECS and custom serializers. The `reason` field is now deprecated.
* **🧐Enhancement** (ES) Users defined with LDAP, external, and `ror_kbn` authentication are no longer treated as local users by the impersonation mechanism
* **🧐Enhancement** (ES) The ROR Kibana plugin can no longer be used when the `prompt_for_basic_auth: true` setting is configured
* **🐞Fix** (KBN) Resolved a memory leak related to direct calls via the Kibana API
* **🐞Fix** (KBN) No longer shows the "Data Set Quality" and "Index management" applications to users with RO or RO_strict access
* **🐞Fix** (KBN) Fixed JWT token authorization when using embedded Kibana
* **🐞Fix** (KBN) Fixed the styling of the page-not-found screen for Kibana 9.x
* **🐞Fix** (KBN) Correctly displays the "Who uses what indices?" Audit Dashboard visualization when indices are not specified in the audit events
* **🐞Fix** (ES) [Improved stability when sending audit logs to another cluster, so temporary remote cluster outages no longer affect the main cluster](https://forum.readonlyrest.com/t/sending-logs-to-another-cluster/2925)
* **🐞Fix** (ES) Fixed Search Profiler being inactive in Kibana 8.18.0+
* **🐞Fix** (ES) `beshultd/elasticsearch-readonlyrest` images for ES 7.16.x, 7.17.0–7.17.6, and 8.0.x–8.4.x now ship with a patched JDK, replacing bundled JDK 17.0.0–17.0.4 / JDK 18, which crashes on cgroup v2 hosts due to JDK-8287073

### (2026-05-11) What's new in **ROR 1.68.0**
* **🐞Fix** (KBN) Fixed `jsonwebtoken-ancient` being stripped from Kibana builds earlier than 7.11.0 or smth like that

### (2026-05-11) What's new in **ROR 1.67.2**
* **🚀New** (KBN) 9.2.1, 9.1.7, 8.19.7 support
* **🚀New** (ES) 9.2.1, 9.1.7, 8.19.7 support

### (2026-05-11) What's new in **ROR 1.67.1**
* **🚀New** (ES) 9.2.0, 9.1.6, 8.19.6 support
* **🧐Enhancement** (ES) Allow using the `actions` rule with the `kibana` rule in the same block when `kibana.access: unrestricted`
* **🐞Fix** (KBN) Fixed JWT handling for wrong license edition
* **🐞Fix** (KBN) Suppressed “Forbidden” toast in Discover/Dashboard on Kibana 8.x–9.x
* **🐞Fix** (KBN) [Resolved report download failure on Kibana 9.1.x](ttps://forum.readonlyrest.com/t/unable-to-download-reports-from-kibana/2859/2)
* **🐞Fix** (KBN) Fixed timeout when saving Security settings
* **🐞Fix** (KBN) Restored visibility of reports when multiple data streams exist for a reporting index
* **🐞Fix** (KBN) Fixed invisible reports for non-tenancy users on Kibana 9.1.x

### (2022-06-21) What's new in **ROR 1.41.0**
* **🚀New** (ES) Added groups_and mode to [ror_kbn_auth](https://docs.readonlyrest.com/elasticsearch#ror_kbn_auth) and [jwt_auth](https://docs.readonlyrest.com/elasticsearch#jwt_auth) rules
* **🧐Enhancement** (KBN) Prevent native credentials dialogue to appear in Kibana when ES responds 401
* **🐞Fix** (ES|KBN) tenancy selector didn't work well with jwt_auth and ror_kbn_auth rules
* **🐞Fix** (KBN) OIDC connector not working in Kibana < 7.12.0

