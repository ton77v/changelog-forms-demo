# Changelog

<!-- AUTO-GENERATED from changelog/*.yaml — do not edit directly -->

### (2026-05-11) What's new in **ROR 1.69.1**
* **🚨Security Fix** (KBN) Fixed vulnerability [CVE-2026-2950](https://nvd.nist.gov/vuln/detail/CVE-2026-2950)
* **🚀New** (KBN) 9.4.0 9.3.4, 9.3.3, 9.2.8, 8.19.15, 8.19.14 support
* **🚀New** (ES) 9.3.4, 9.3.3, 9.2.8, 8.19.15, 8.19.14 support
* **🚀New** (ECK) 3.4.0 support
* **🐞Fix** (KBN) Fixed `jsonwebtoken-ancient` being stripped from Kibana builds earlier than 7.11.0

### (2026-05-11) What's new in **ROR 1.68.0**
* **🐞Fix** (KBN) Fixed `jsonwebtoken-ancient` being stripped from Kibana builds earlier than 7.11.0 or smth like that

### (2026-05-11) What's new in **ROR 1.67.2**
* **🚀New** (KBN) 9.2.1, 9.1.7, 8.19.7 support
* **🚀New** (ES) 9.2.1, 9.1.7, 8.19.7 support

### (2022-06-21) What's new in **ROR 1.41.0**
* **🚀New** (ES) Added groups_and mode to [ror_kbn_auth](https://docs.readonlyrest.com/elasticsearch#ror_kbn_auth) and [jwt_auth](https://docs.readonlyrest.com/elasticsearch#jwt_auth) rules
* **🧐Enhancement** (KBN) Prevent native credentials dialogue to appear in Kibana when ES responds 401
* **🐞Fix** (ES|KBN) tenancy selector didn't work well with jwt_auth and ror_kbn_auth rules
* **🐞Fix** (KBN) OIDC connector not working in Kibana < 7.12.0

