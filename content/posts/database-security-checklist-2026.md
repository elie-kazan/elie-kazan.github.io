---
title: "Database Security Checklist for 2026"
date: 2025-12-10
description: "The essential security hardening steps I apply to every new database deployment."
emoji: "🛡️"
---

Every time I set up a new database, I run through this checklist. It's not exhaustive, but it covers the fundamentals that prevent the most common attack vectors.

## Authentication & Access

**Disable default accounts.** The `postgres` superuser should not be used for application connections. Create dedicated roles with the minimum privileges needed.

**Enforce strong passwords** — or better yet, use certificate-based authentication. If you're on AWS, IAM database authentication eliminates passwords entirely for supported engines.

**Restrict network access.** Your database should not be reachable from the public internet. Use `pg_hba.conf` (PostgreSQL) or bind-address configs to limit connections to known application servers and admin IPs only.

## Encryption

**Encrypt in transit.** TLS should be mandatory for all connections. In PostgreSQL, set `ssl = on` and configure `ssl_min_protocol_version = 'TLSv1.3'`.

**Encrypt at rest.** Use your cloud provider's native encryption (AWS RDS encryption, GCP CMEK, etc.) or LUKS for self-managed instances.

## Monitoring & Auditing

**Enable query logging** — but be smart about it. Log all DDL statements and failed authentication attempts. For DML, use `pgaudit` to selectively log access to sensitive tables without drowning in noise.

**Set up alerts** for anomalies: unusual connection counts, queries from unexpected IPs, privilege escalation attempts.

## Backups & Recovery

**Test your backups.** A backup that hasn't been restored is just a hope. I schedule monthly restore drills to a separate environment and verify data integrity.

**Encrypt your backups.** A database backup contains everything an attacker wants. Encrypt at rest and in transit, and rotate encryption keys.

## Keeping Up to Date

**Patch regularly.** Subscribe to security mailing lists for your database engine. CVEs get disclosed regularly, and unpatched databases are low-hanging fruit.

This checklist is my baseline. Your specific environment may need additional hardening, but if you cover these fundamentals, you're ahead of most deployments I've audited.
