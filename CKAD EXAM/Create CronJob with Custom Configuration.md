## Problem: Scheduled Jobs with Specific Requirements

Create a CronJob that:

- Runs every Monday
- Completes 5 times (completions: 5)
- Runs 3 pods in parallel (parallelism: 3)
- Keeps 5 successful job history
- Keeps 10 failed job history
- Terminates after 20 seconds (activeDeadlineSeconds)

## Solution: Create CronJob

### Step 1: Create CronJob with basic settings

```bash
kubectl create cronjob weekly-report \
  --image=busybox \
  --schedule="0 0 * * 1" \
  --dry-run=client -o yaml > cronjob.yaml \
  -- /bin/sh -c "echo 'Running weekly report'; sleep 5"
```

This generates:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: weekly-report
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: weekly-report
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - echo 'Running weekly report'; sleep 5
            image: busybox
            name: weekly-report
            resources: {}
          restartPolicy: OnFailure
  schedule: 0 0 * * 1
status: {}
```

### Step 2: Edit to add all requirements

```bash
vim cronjob.yaml
```

Add completions, parallelism, history limits, and timeout:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: weekly-report
spec:
  schedule: "0 0 * * 1"  # Every Monday at midnight
  successfulJobsHistoryLimit: 5   # Keep 5 successful jobs
  failedJobsHistoryLimit: 10      # Keep 10 failed jobs
  jobTemplate:
    spec:
      completions: 5                # Complete 5 times
      parallelism: 3                # Run 3 pods in parallel
      activeDeadlineSeconds: 20     # Terminate after 20 seconds
      template:
        spec:
          containers:
          - name: weekly-report
            image: busybox
            command:
            - /bin/sh
            - -c
            - echo 'Running weekly report'; sleep 5
          restartPolicy: OnFailure
```

### Step 3: Apply the CronJob

```bash
kubectl apply -f cronjob.yaml
```

### Step 4: Verify CronJob

```bash
kubectl get cronjob
```

Output:

```
NAME            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
weekly-report   0 0 * * 1     False     0        <none>          10s
```

Describe for details:

```bash
kubectl describe cronjob weekly-report
```

### Step 5: Test immediately (don't wait for Monday)

Create a Job from the CronJob:

```bash
kubectl create job weekly-report-test --from=cronjob/weekly-report
```

Watch the job:

```bash
kubectl get jobs -w
```

Check pods created:

```bash
kubectl get pods -l job-name=weekly-report-test
```

Output:

```
NAME                       READY   STATUS      RESTARTS   AGE
weekly-report-test-abc12   0/1     Completed   0          15s
weekly-report-test-def34   0/1     Completed   0          15s
weekly-report-test-ghi56   0/1     Completed   0          10s
weekly-report-test-jkl78   0/1     Completed   0          10s
weekly-report-test-mno90   0/1     Completed   0          5s
```

Check logs:

```bash
kubectl logs job/weekly-report-test
```

### Step 6: Verify history limits

After multiple runs, check job history:

```bash
kubectl get jobs
```

You should see only 5 successful and 10 failed jobs retained.

## CronJob Schedule Format

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
│ │ │ │ │
│ │ │ │ │
* * * * *
```

Common schedules:

```bash
"0 0 * * 1"      # Every Monday at midnight
"*/5 * * * *"    # Every 5 minutes
"0 9 * * 1-5"    # Weekdays at 9 AM
"0 0 1 * *"      # First day of month at midnight
"0 */2 * * *"    # Every 2 hours
```