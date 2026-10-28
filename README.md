# Cloud SQL PostgreSQL – Storage Shrink Guide

## Overview

This document provides step-by-step commands to reduce the provisioned
SSD storage of an existing Google Cloud SQL PostgreSQL instance.

The storage shrink operation allows disk optimization without manually
creating a new Cloud SQL instance.

## Prerequisites

- Google Cloud SDK (gcloud CLI) installed and authenticated.
- Required Cloud SQL administrative permissions.
- Existing Cloud SQL PostgreSQL instance.
- Sufficient free disk space for the shrink operation.
- Verified database backup.
- Approved maintenance window.

## 1. Check Current Storage

Run the following command to check the current instance configuration.

```bash
gcloud sql instances describe INSTANCE_NAME \
  --project=PROJECT_ID \
  --format="yaml(name,state,settings.dataDiskSizeGb,settings.dataDiskType)"
```

## 2. Check Minimum Allowed Storage Size

Before reducing storage, check the minimum supported target size.

```bash
gcloud sql instances get-storage-shrink-config INSTANCE_NAME \
  --project=PROJECT_ID
```

Example output:

```yaml
kind: sql#getDiskShrinkConfig
minimalTargetSizeGb: '61'
```

Important:

- The target storage must not be smaller than `minimalTargetSizeGb`.
- Minimum allowed storage is not necessarily the recommended target.
- Google recommends reserving additional capacity for operations
  such as reindexing and database maintenance.
- Shrinking storage may negatively impact IOPS.

## 3. Shrink Existing Instance Storage

Run the following command to reduce provisioned storage.

```bash
gcloud sql instances perform-storage-shrink INSTANCE_NAME \
  --project=PROJECT_ID \
  --storage-size=TARGET_SIZE_GB
```

Replace:

- INSTANCE_NAME: Existing Cloud SQL instance name.
- PROJECT_ID: Google Cloud project ID.
- TARGET_SIZE_GB: Desired storage size in GB.

Note:

- The operation modifies the existing instance.
- No manual creation of a new instance is required.
- Downtime may occur during the shrink operation.
- Always verify backup availability before execution.

## 4. Verify Storage After Completion

```bash
gcloud sql instances describe INSTANCE_NAME \
  --project=PROJECT_ID \
  --format="yaml(name,state,settings.dataDiskSizeGb)"
```

Verify that:

- Instance state is RUNNABLE.
- Storage size matches the requested target.
- Databases and applications are accessible.
- Database health checks pass.

## 5. Example – CSU DR Test Instance

Project: csu-global-dev

Instance: dr-test-postgresql-csu-dev

Region: us-east1

PostgreSQL Version: 17.11

Cloud SQL Edition: Enterprise

### Before Configuration

| Configuration | Value |
|---|---|
| Storage | 578 GB |
| Storage Type | SSD |
| Minimum Allowed Storage | 61 GB |
| Target Storage | 61 GB |
| Potential Storage Reduction | 517 GB |
| Potential Reduction Percentage | 89.4% |
| Estimated Operation Time | 20 minutes |

### Check Minimum Storage

```bash
gcloud sql instances get-storage-shrink-config dr-test-postgresql-csu-dev \
  --project=csu-global-dev
```

### Perform Storage Shrink

Execute only after confirming sufficient capacity and backup readiness.

```bash
gcloud sql instances perform-storage-shrink dr-test-postgresql-csu-dev \
  --project=csu-global-dev \
  --storage-size=61
```

### Verify Updated Storage

```bash
gcloud sql instances describe dr-test-postgresql-csu-dev \
  --project=csu-global-dev \
  --format="yaml(name,state,settings.dataDiskSizeGb)"
```

Expected output after successful completion:

```yaml
name: dr-test-postgresql-csu-dev
state: RUNNABLE
settings:
  dataDiskSizeGb: '61'
```

## 6. Important Notes

1. The standard `gcloud sql instances patch --storage-size`
   command cannot reduce existing disk size.

2. Use `get-storage-shrink-config` to determine the minimum
   supported storage size.

3. Use `perform-storage-shrink` to perform the supported
   storage reduction operation.

4. The minimum supported target may change based on actual
   disk usage and instance configuration.

5. Storage shrink operations may cause downtime.

6. Reducing SSD storage can affect available IOPS.

7. Always take and verify a backup before shrinking.

8. Validate database connectivity and application health
   after the operation.

9. Confirm actual cost savings through Google Cloud Billing.

## 7. Official Documentation

Google Cloud SQL Storage Shrink:

https://cloud.google.com/sql/docs/postgres/shrink-instance-storage

Google Cloud SQL Quotas and Limits:

https://cloud.google.com/sql/docs/quotas

---

Maintained by: DevOps Team

Purpose: Cloud SQL PostgreSQL Storage and Cost Optimization
