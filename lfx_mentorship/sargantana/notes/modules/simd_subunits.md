# SIMD / Vector sub-units

Directory: `rtl/datapath/rtl/exe_stage/rtl/simd/`

These modules are the leaf datapaths instantiated inside [[simd_unit]] and
[[vagu]]. They are listed here for reference; deep dives are out of scope for the
top-level notes.

| File                                                             | Role                                           |
| ---------------------------------------------------------------- | ---------------------------------------------- |
| `functional_unit.sv`                                             | common vector functional-unit wrapper/dispatch |
| `vaddsub.sv`                                                     | vector add/subtract                            |
| `vsaaddsub.sv`                                                   | vector saturating add/subtract                 |
| `vwaddsub.sv`                                                    | vector widening add/subtract                   |
| `vmul.sv`                                                        | vector multiply                                |
| `vsmul.sv`                                                       | vector saturating multiply                     |
| `vdiv.sv`, `div_2bits.sv`                                        | vector divide                                  |
| `vshift.sv`                                                      | vector shifts                                  |
| `vnclip.sv`                                                      | vector narrowing clip                          |
| `vcomp.sv`, `comp_f32.sv`, `comp_f64.sv`                         | vector comparisons                             |
| `vredtree.sv`, `vfredtree.sv`, `vfredoladder.sv`                 | reduction trees                                |
| `vpopc.sv`, `vcpop.sv`, `viota.sv`, `vfirst.sv`, `vmsb_i_o_f.sv` | mask/prefix ops                                |
| `vczeros.sv`, `vbrev.sv`                                         | count-zeros / bit-reverse                      |
| `maxmin_f32.sv`, `maxmin_f64.sv`                                 | vector FP min/max                              |
| `fp16_to_fp32.sv`, `fp32_to_fp64.sv`, `conv_zvfhmin.sv`          | FP conversions                                 |
| `vf7_mf.sv`, `vf7_wrapper.sv`, `vfpu_drac_wrapper.sv`            | vector FPU                                     |
| `pending_vfp_ops_queue.sv`                                       | in-flight vector-FP op tracking                |

## Related

[[simd_unit]] · [[vagu]] · [[exe_stage]] · [[pipeline_overview]]
