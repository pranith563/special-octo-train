1. For each of the 18 corresponding convolution constants, compare 2.28 versus 2.48.1:
   - stored datatype
   - dimensions and axis order
   - raw byte length and SHA-256
   - integer min/max and zero percentage
   - first 32 values
   - dequantized min/max using its scale and offset
2. Dump CPU intermediate outputs from both versions and find the first tensor where 2.48.1 becomes zero. Start with the first convolution output.
3. Make a surgical 2.48.1 converter experiment that changes only those 18 is_overridden values immediately before quantization/serialization. Then rerun CPU inference. Do not change calibration, per-channel mode, I/O, or optimization settings.
4. Compare the operation inventory and producer/consumer wiring responsible for the additional 100 tensors.
