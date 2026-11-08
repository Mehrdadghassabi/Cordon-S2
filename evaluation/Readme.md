| Metric | Single-record formula |
|---|---|
| **ASR** | `Attack_success` |
| **PDR** | `Poison_detected` |
| **CR** | `Poison_detected × Attack_success`, if `Knowledge = 0`; otherwise `N/A`. |
| **CCR** | `Attack_success`, if `Knowledge = 0 AND Poison_detected = 1`; otherwise `N/A`. |
| **LR** | ` Contaminated × Knowledge` |
