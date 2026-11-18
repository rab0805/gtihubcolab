# Rule R1 verification results

Calculated by calling `pensionableSalary` in `api/src/domain/pension.ts` with the default scheme.

| # | fullTimeSalary | partTimeFactor | expected | actual | matches | why it is an edge case |
|---|---:|---:|---|---|---|---|
| 1 | 17545 | 1 | `0` | `0` | ✅ | Salary is exactly at the franchise. |
| 2 | 17545.01 | 0.5 | `0.01` | `0` | ❌ | Tests half-cent rounding after the part-time adjustment. |
| 3 | 200000 | 0.5 | `60127.5` | `60127.5` | ✅ | Salary exceeds the cap; verifies the cap is applied before the factor. |
| 4 | -1 | 0.5 | `RangeError: fullTimeSalary must be a non-negative number` | `RangeError: fullTimeSalary must be a non-negative number` | ✅ | Negative salary is invalid. |
| 5 | 50000 | 0 | `RangeError: partTimeFactor must be greater than 0 and at most 1` | `RangeError: partTimeFactor must be greater than 0 and at most 1` | ✅ | A zero part-time factor is invalid. |

**Score: 4/5 match.**

Row 2 returns `0` because binary floating-point subtraction produces a value slightly below `0.01`. After multiplication by `0.5`, the amount is slightly below half a cent and rounds down to zero.

## Validation

Command run successfully (exit code 0):

```shell
node -e "import('./api/src/domain/pension.ts').then(m=>{for (const [s,f] of [[17545,1],[17545.01,0.5],[200000,0.5],[-1,0.5],[50000,0]]) {try{console.log(s,f,m.pensionableSalary(s,f))}catch(e){console.log(s,f,e.name+': '+e.message)}}})"
```

Output:

```text
17545 1 0
17545.01 0.5 0
200000 0.5 60127.5
-1 0.5 RangeError: fullTimeSalary must be a non-negative number
50000 0 RangeError: partTimeFactor must be greater than 0 and at most 1
```
