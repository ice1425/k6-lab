# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=288.34ms, min=288.34ms, med=288.34ms, max=288.34ms, p(90)=288.34ms, p(95)=288.34ms. All values are the same because the run made only one request (1 VU, 1 iteration). The check `status is 200` passed (1 out of 1).
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 77 requests at 3.64 requests/s, p(95) = 284.83 ms. The rate stays below 5 because one iteration of a VU is not just one request. Each VU sends the request (about 0.28 s), then sleeps for 1 s, so one iteration takes about 1.28 s (iteration_duration med = 1.27 s). One VU therefore makes only about 1 / 1.28 ≈ 0.78 requests/s, and 5 VUs make about 3.9 requests/s. The measured rate is a little lower again (3.64/s) because the first iteration of each VU is slower while it opens the connection (iteration_duration max = 2.27 s), and K6 divides by the whole run time of 21.1 s rather than 20 s.
- Step 0.6, exit code of the normal run and of the run with P95=300: The normal run passed all three thresholds (checks rate = 100.00%, p(95) = 394.72 ms < 1500 ms, http_req_failed = 0.00%) and the exit code was 0. With P95=300 the p(95) was 392.87 ms, so the rule p(95)<300 failed, K6 printed that the thresholds on http_req_duration had been crossed, and the exit code was 99. All checks still passed in that run, which shows that a threshold can fail a test even when every response is correct.
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: The script uses 3 VUs for 30 s with the thresholds http_req_duration p(95)<1000 and p(99)<1500, and http_req_failed rate<0.01. All rules passed: p(95) = 385.86 ms, p(99) = 395.25 ms and http_req_failed = 0.00%, with exit code 0. I then changed the requirement to "95% of requests within 300 ms" (run with -e P95=300). The p(95) rule failed with p(95) = 388.03 ms, the p(99) and failure rules still passed, and the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it: The request sent excludedIngredients ['pepperoni'] and excludedTools ['knife'] in lower case, but explore.js showed that the API names them "Pepperoni" and "Knife". The API matches names case-sensitively, so it found nothing to exclude and ignored both restrictions, while still answering with status 200. With 5 VUs for 20 s, every request returned 200, but the check "no pepperoni" failed 14 of 75 times (81% passed) and "no knife needed" failed 29 of 75 times (61% passed). After changing the request to ['Pepperoni'] and ['Knife'], all checks passed (100%, 222 out of 222). The checks under load found the bug because the pizza is chosen at random: a single manual request can by chance return a pizza without pepperoni that needs no knife and look correct, but 75 requests with a check on every answer made the wrong pizzas show up as a clear failure rate.

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     | 0.51  | 421.32 ms | 553.53 ms | 0.00% | 100.00% |
| A2   | Average load | 3     | 1.29  | 370.39 ms | 395.74 ms | 0.00% | 100.00% |
| A3   | Stress       | 5     | 2.09  | 392.85 ms | 2.22 s    | 0.00% | 100.00% |
| A4   | Spike        | 5     | 1.01  | 386.62 ms | 712.34 ms | 0.00% | 100.00% |
| A5   | Soak         | 2     | 0.99  | 380.22 ms | 621 ms    | 0.00% | 100.00% |

## Part B

1. **Results table for A1–A5**
   - The table above is filled in with the numbers from my runs.
   - Every run passed all of its thresholds, with 0.00% failed requests and 100.00% checks.

2. **Why 5 VUs made fewer than 5 requests per second (Step 0.2)**
   - One iteration of a VU = one request (about 0.28 s) + sleep 1 s = about **1.28 s per iteration**.
   - So one VU makes only 1 ÷ 1.28 ≈ **0.78 requests/s**.
   - Five VUs make about 5 × 0.78 ≈ **3.9 requests/s**, not 5.
   - The measured rate was a little lower again (**3.64/s**, 77 requests) because:
     - the first iteration of each VU was slower while it opened the connection (iteration_duration max = 2.27 s), and
     - K6 divides by the real run time of 21.1 s, not 20 s.

3. **Did the response time grow from A2 (3 users) to A3 (5 users)?**
   - **Only slightly.** p(95) went from **370.39 ms** to **392.85 ms** (about 22 ms, or 6%).
   - Per group: home 280.57 → 306.89 ms, recommendation 384.8 → 403.97 ms.
   - The request rate grew from 1.29 to 2.09/s, almost in proportion to the users (5/3 ≈ 1.67×), and no request failed.
   - The max of 2.22 s in A3 was **one single** slow home page request. The p(90) of the home page stayed at 290.73 ms.
   - **Operational envelope:** QuickPizza is built for K6 learners around the world to use at the same time. Our 5 users are still **far inside the envelope**, nowhere near the edge. A 22 ms difference is about the same as normal internet variation, and these runs cannot show where the edge is.

4. **Spike test: how fast did the response time return to normal?**
   - In the VUs graph, users jumped from 1 to 5 at about 15:45:10 and fell back to 1 at about 15:45:50.
   - Before the spike: p(95) about 330–385 ms.
   - During the spike: p(95) about 365–385 ms → **almost no change**.
   - End of the spike: about 340 ms → **back to normal within one sampling interval (about 10 s)**.
   - The highest points in the graph (about 425 ms at 15:45:57 and 450 ms at the end) came when **only 1 user** was running. Each 10-second interval then holds very few requests, so one slow request raises the p(95) of the whole interval. This is noise, not slow recovery.

5. **Soak test: did it get worse over time?**
   - **No.**
   - First 2 minutes: p(95) mostly about 345–390 ms.
   - Last 2 minutes: p(95) mostly about 320–390 ms.
   - There were three isolated peaks in the middle (about 435, 490 and 460 ms), but each fell back right away. There was no upward trend.
   - The request rate stayed at about 0.9–1.2/s, and the error rate was 0.00% for the whole run (0 of 657 requests).
   - **A defect a soak test of several hours could reveal:** a **memory leak**, where the server keeps a little memory after every request and never frees it, so the system slows down over time until it crashes. Other examples are database connections or file handles that are never closed, and logs or caches that grow until the disk or memory is full.

6. **Why the thresholds use p(95) instead of the average**
   - An average **hides the slowest users.** A few slow requests barely move the average, because many fast requests cover them up.
   - Examples: in A3 the average was 323.3 ms, but the slowest request took 2.22 s. In A4 the recommendation average was 362.8 ms, but its p(95) was 458.44 ms.
   - p(95) means 95% of requests were faster than this value, so it reflects what almost every user experiences, including the slower ones.
   - It matches how real requirements are written, e.g. "95% of requests within 1 second".
   - It is more stable than the max, which can come from one unlucky request, like the 2.22 s in A3.

7. **Two reasons a lab test could give different numbers**
   - **Different network path:** our requests travelled from Bangkok over the public internet to the server. Most of the roughly 270 ms minimum response time is network latency, and it varies with internet traffic. In a lab, the load generator and the system are usually on the same local network with about 1 ms of latency, so times would be much lower and more stable.
   - **Shared server outside our control:** people around the world use QuickPizza at the same time as us, so their traffic mixes with ours and may explain the isolated peaks we saw. Grafana may also use caching, a CDN or rate limits that a lab system does not have. In a lab, the hardware, configuration and other traffic are known and controlled, so results differ and are easier to repeat.

8. **Why never run large stress, spike or breakpoint tests against a site you do not own**
   - In practice it is a **denial-of-service attack.**
   - It can slow down or crash the site for real users, and it costs the owner money for extra servers and bandwidth.
   - Security systems may block your network.
   - Without permission it is unethical and can be illegal, for example under Thailand's Computer-Related Crime Act.
   - It is also a useless test: you do not know the system's normal load, you cannot see its servers, and you may only measure the limit of your own internet connection.
   - These tests belong in a test environment that you own or have written permission to test.

9. **A requirement for an app I use every day (a food delivery app)**
   - **Vague:** "Searching for restaurants should be fast."
   - **Testable:** "With 200 users searching for restaurants at the same time for 10 minutes, the search API answers 95% of requests within 800 ms and 99% within 1.5 seconds, fewer than 0.5% of requests fail, and more than 99% of search results contain at least one restaurant."
   - **K6 thresholds** (to run only against a test environment of the app, never the real service):

   ```javascript
   export const options = {
     vus: 200,
     duration: '10m',
     thresholds: {
       'http_req_duration{group:::search}': ['p(95)<800', 'p(99)<1500'],
       http_req_failed: ['rate<0.005'],
       checks: ['rate>0.99'],
     },
   };
   ```