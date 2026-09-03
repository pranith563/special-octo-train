Stop testing calibration variants. Keep the exact model, input and settings fixed.

1. Verify tool matching
- Inspect the 2.28 context only with the 2.28 qnn-context-binary-utility.
- Inspect the 2.48.1 context only with the 2.48.1 utility.
- Run each context with its matching qnn-net-run and HTP libraries.
- Record utility version, context buildId, context SHA256 and runtime version.

2. Validate the alleged corrupt encodings
For every graph input and output report:
- dataType
- quantizeParams.definition
- quantizationEncoding
- scale
- offset

Do not interpret scale or offset unless definition is DEFINED and encoding is SCALE_OFFSET.

3. Compare pre-context converter output
From both generated *_net.json files report:
- number of tensors
- count with is_overridden=true
- count of DEFINED encodings
- count of scale == 0
- count of non-finite scales
- count of abs(scale) > 1e6
- input/output datatype and encoding
- weight encoding min/max scale range

Do not report confidential tensor names; hash names if identification is needed.

4. Isolate the failing stage using QAIRT 2.48.1
Run the generated source-native QNN model library before creating a context:
A. Host qnn-net-run with QNN CPU
B. Android model library with HTP online graph preparation
C. Offline SM8850 HTP context

Use the same logical float32 input and native uint8 output for all three.

Interpretation:
- CPU already zeros: TensorFlow converter/quantizer problem.
- CPU works, online HTP zeros: HTP lowering/runtime problem.
- CPU and online HTP work, offline context zeros: context-binary generation problem.
- All three work but application fails: application buffer integration problem.

5. For every execution return:
- exit code
- output byte count
- SHA256
- non-zero count
- min/max/mean
- first 32 raw uint8 values

6. Repeat the same A/B/C matrix with 2.28 using identical converter options.

Do not use ignore_encodings, per-channel overrides, different calibration data, or float fallback during this isolation.
