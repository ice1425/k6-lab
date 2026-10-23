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
| A3   | Stress       | 5     |       |       |     |        |        |
| A4   | Spike        | 5     |       |       |     |        |        |
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B

1.
2.
3.
4.
5.
6.
7.
8.
9.