# Personal data

Not legal advice; the law's text is linked so the user and their DPO can read it.

## What counts

- **Personal data** (LGPD art. 5 I; GDPR art. 4(1)): information about an identified or identifiable natural person.
  Names, CPF, RG, email, phone, address, customer IDs that a lookup can resolve, device IDs, and location all qualify.
- **Sensitive data** (LGPD art. 5 II): racial or ethnic origin, religious conviction, political opinion, union or
  religious, philosophical, or political organization membership, health or sex life, genetic or biometric data,
  when linked to a person. GDPR art. 9 special categories are close to this list.
- **Anonymised data** is outside the law only while the anonymisation cannot be reversed with reasonable means
  (LGPD art. 12).
- **Pseudonymised data stays personal data**: a hash or token that the controller can link back with information kept
  separately (LGPD art. 13 §4; GDPR art. 4(5)). A plain hash of a CPF or an email is pseudonymisation, since the space of
  values can be enumerated.
- https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm · https://gdpr-info.eu/art-4-gdpr/ ·
  https://gdpr-info.eu/art-9-gdpr/

## What the principles ask of a pipeline

- **Purpose and necessity** (LGPD art. 6 I and III; GDPR art. 5(1)(b) and (c)): carry only the columns the stated use
  needs. A gold table for a dashboard of counts carries no names or IDs.
- **Security** (art. 6 VII): masking, access policies, and no personal data in logs, samples, or exports.
- **Rights** (LGPD art. 18; GDPR art. 17): correction and deletion must reach every copy, so the pipeline knows where
  each person's rows land, and deletes propagate through the layers (time travel and backups included in the plan).

## Incidents

- LGPD art. 48 with ANPD Resolution CD/ANPD 15/2024: a relevant incident is reported to the ANPD within **3 business
  days** of learning it affected personal data, and to the affected people in the same period; small processing agents
  get double. Every incident is recorded for 5 years, reported or not.
  https://www.gov.br/anpd/pt-br/canais_atendimento/agente-de-tratamento/comunicado-de-incidente-de-seguranca-cis
- GDPR art. 33: the authority within 72 hours. https://gdpr-info.eu/art-33-gdpr/
- A dataset with personal data sent to the wrong place (a public bucket, a chat, a public repo) is an incident: tell the
  user at once.

## Masking by platform

- **Databricks Unity Catalog**: row filters and column masks are SQL UDFs applied at query time
  (`ALTER TABLE … SET ROW FILTER`, `ALTER TABLE … ALTER COLUMN … SET MASK`); for many tables, ABAC policies on governed
  tags. https://docs.databricks.com/aws/en/tables/row-and-column-filters
- **Snowflake** (Enterprise Edition and up): masking policies by role, applied wherever the column appears
  (`CREATE MASKING POLICY …`, `ALTER TABLE … MODIFY COLUMN … SET MASKING POLICY`).
  https://docs.snowflake.com/en/user-guide/security-column-ddm-intro
- **BigQuery**: policy tags gate column reads; data policies mask at query time (hash, last four, email mask,
  nullify, default value). https://docs.cloud.google.com/bigquery/docs/column-data-masking-intro
- **Anywhere else**: drop the column, aggregate, or replace it with a keyed token in the silver layer, keeping the
  key outside the analytics store.
