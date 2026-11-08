| Metric | Single-record formula |
|---|---|
| **ASR** | `Attack_success` |
| **PDR** | `Poison_detected` |
| **CR** | ` Attack_success × Poison_detected`, if `Knowledge = 1`; otherwise `N/A`. |
| **CCR** | `Attack_success`, if `Knowledge = 1 AND Poison_detected = 1`; otherwise `N/A`. |
| **LR** | ` Contaminated × Knowledge` |
