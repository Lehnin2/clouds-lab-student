# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
49.99
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->256.2<!--/answer--> | <!--answer:vm_warm_median_ms-->210.2<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->3005.1<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->138.4<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1357.1<!--/answer--> | <!--answer:function_warm_median_ms-->634.5<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
```text
Warm request median latency (lower is better)

VM             210.2 ms | #############
Cloud Run      138.4 ms | #########
Cloud Function 634.5 ms | ########################################
```
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
On the VM, 1, 10, or 100 concurrent requests all reach the same e2-micro machine. No additional VMs are created automatically; performance is limited by its CPU and memory. Cloud Run automatically starts container instances as traffic increases, but our deployment is capped at three instances by --max-instances=3. Cloud Functions Gen2 also creates instances automatically as needed. Our function has concurrency set to one request per instance, so simultaneous requests may require additional instances or queueing. Its scaling is subject to Google Cloud quotas and configured limits.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
Deploying on the VM required the most manual work: creating a custom VPC and subnet, configuring firewall rules, provisioning an e2-micro instance, installing Docker, pulling the image, and starting the container. We must also maintain the VM and its operating system. Cloud Run was simpler: after building and pushing the Docker image to Artifact Registry, we deployed it with one command and configured its port, public access, and scaling limits. Google manages the servers. Cloud Functions Gen2 required only the Python source, runtime, entry point, and HTTP trigger. Google handled building and deploying it. However, IAM permissions still needed configuration.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
```text
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-332499246111.us-central1.run.app
LAST DEPLOYED BY: service-332499246111@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-27T18:02:07.070203Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-332499246111.us-central1.run.app
LAST DEPLOYED BY: mohammed.Silini@eurecom.fr
LAST DEPLOYED AT: 2026-10-27T17:53:03.879120Z
```
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows two Cloud Run services: echo-bot, which we deployed directly, and echo, which was created automatically when we deployed our second-generation Cloud Function. The echo service was deployed by Google's Cloud Functions service agent rather than our user account. This demonstrates that Cloud Functions Gen2 uses Cloud Run as its underlying execution infrastructure. Both can start container instances on demand and scale down when idle, so both may experience cold-start latency. In our measurements, Cloud Run's first request took 3005.1 ms and the Function's took 1357.1 ms. These are different individual measurements, not proof that one platform always starts faster. Cloud Run gives us direct control over container deployment and configuration, whereas Cloud Functions provides a simpler source-code-and-entry-point deployment interface.
<!--/answer-->
