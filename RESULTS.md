# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=288.34ms, min=288.34ms, med=288.34ms, max=288.34ms, p(90)=288.34ms, p(95)=288.34ms. All values are the same because the run made only one request (1 VU, 1 iteration). The check `status is 200` passed (1 out of 1).
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 77 requests at 3.64 requests/s, p(95) = 284.83 ms. The rate stays below 5 because one iteration of a VU is not just one request. Each VU sends the request (about 0.28 s), then sleeps for 1 s, so one iteration takes about 1.28 s (iteration_duration med = 1.27 s). One VU therefore makes only about 1 / 1.28 ≈ 0.78 requests/s, and 5 VUs make about 3.9 requests/s. The measured rate is a little lower again (3.64/s) because the first iteration of each VU is slower while it opens the connection (iteration_duration max = 2.27 s), and K6 divides by the whole run time of 21.1 s rather than 20 s.
- Step 0.6, exit code of the normal run and of the run with P95=300:
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
- Step 0.8, what was wrong, and how the checks showed it:

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |       |       |     |        |        |
| A2   | Average load | 3     |       |       |     |        |        |
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