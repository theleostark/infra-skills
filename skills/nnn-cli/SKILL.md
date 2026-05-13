---
name: 'Neural Network Node CLI'
description: 'Use when the user needs: Neural Network Node CLI — WebNN API reverse-engineered into a command-line tool. Runs ML inference on ANE/NPU/GPU/CPU from the terminal. Works across the shadow mesh. Use when running on-device inference, benchmarking ANE, building ML pipelines, or routing neural compute across mesh nodes.'
icon: icon.svg
metadata:
  filePattern:
  - '**/nnn*'
  - '**/neural*'
  - '**/webnn*'
  - '**/ane*'
  bashPattern:
  - 'nnn '
  - nnn-cli
  priority: 50
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: nnn-cli
  icon_style: craft-category-glyph-v1
---

# nnn-cli — Neural Network Node

WebNN API concepts mapped to CLI commands. Runs ML inference on Apple ANE, NPU, GPU, or CPU from the terminal.

## Architecture

```
WebNN Browser API          →    nnn-cli Terminal
─────────────────               ─────────────────
navigator.ml               →    nnn context
MLContext                   →    nnn context create --device npu
MLGraphBuilder              →    nnn graph build model.onnx
builder.build()             →    nnn graph compile graph.nn
context.dispatch()          →    nnn run graph.nn --input data.bin
context.readTensor()        →    nnn tensor read output.tensor
MLTensor                    →    nnn tensor create --shape 1,768 --dtype f32
Device selection (npu/gpu)  →    nnn device list | nnn device bench
```

## Commands

### Device Management
```bash
nnn device list              # List available accelerators (ANE, GPU, CPU)
nnn device bench             # Benchmark each device with standard ops
nnn device info              # Show device capabilities + TOPS
nnn device mesh              # Show mesh-wide device map
```

### Context
```bash
nnn context create --device npu     # Create inference context on ANE
nnn context create --device gpu     # Create on GPU
nnn context create --device auto    # Auto-select best device
nnn context destroy                 # Release context
```

### Model Operations
```bash
nnn model load model.onnx           # Load ONNX model
nnn model load model.mlmodel        # Load Core ML model
nnn model load model.mlx            # Load MLX model
nnn model info model.onnx           # Show model architecture + ops
nnn model convert model.onnx --to mlx  # Convert between formats
nnn model quantize model.onnx --to int8 # Quantize model
```

### Graph Building (WebNN operators as subcommands)
```bash
# Build compute graph from operators
nnn graph new mygraph
nnn graph add mygraph matmul --a input --b weights
nnn graph add mygraph relu --input matmul_out
nnn graph add mygraph softmax --input relu_out --axis 1
nnn graph compile mygraph --device npu --output graph.nn
```

### Tensor Operations
```bash
nnn tensor create --shape 1,768 --dtype f32 --device npu
nnn tensor read tensor_id --format json
nnn tensor write tensor_id --data input.bin
nnn tensor info tensor_id
```

### Inference
```bash
# Run inference
nnn run model.onnx --input image.bin --device npu
nnn run model.onnx --input "text prompt" --device auto
nnn run graph.nn --input data.bin --output result.bin

# Batch inference
nnn batch model.onnx --input-dir ./images/ --device npu --parallel 4

# Stream inference
echo "hello" | nnn stream model.onnx --device npu
```

### Mesh Routing
```bash
# Route inference across shadow mesh
nnn mesh route model.onnx --input data.bin
# Auto-selects best node: JARVIS (38 TOPS ANE) > NOVA (30 TOPS NPU) > AURION (CPU)

nnn mesh bench --model model.onnx  # Benchmark model across all mesh nodes
nnn mesh status                     # Show mesh neural compute capacity
```

## WebNN Operator → CLI Mapping

| WebNN Operator | nnn-cli | Backend |
|---|---|---|
| `relu()` | `nnn op relu` | ANE native |
| `sigmoid()` | `nnn op sigmoid` | ANE native |
| `softmax()` | `nnn op softmax` | ANE native |
| `conv2d()` | `nnn op conv2d` | ANE native |
| `matmul()` | `nnn op matmul` | ANE/GPU |
| `layerNormalization()` | `nnn op layernorm` | ANE native |
| `gelu()` | `nnn op gelu` | ANE native |
| `batchNormalization()` | `nnn op batchnorm` | ANE/GPU |
| `averagePool2d()` | `nnn op avgpool` | ANE native |
| `gather()` | `nnn op gather` | CPU fallback |
| `reshape()` | `nnn op reshape` | Zero-copy |
| `transpose()` | `nnn op transpose` | ANE/GPU |
| `concat()` | `nnn op concat` | ANE native |
| `split()` | `nnn op split` | ANE native |

## Implementation Stack

### macOS (Apple Silicon)
- Primary: **MLX** (`pip install mlx`) — Apple's native ML framework
- Secondary: **Core ML** via `coremltools`
- Fallback: **ONNX Runtime** with CoreML execution provider

### Linux (AURION/ULTRON)
- Primary: **ONNX Runtime** with CPU/CUDA provider
- Secondary: **llama.cpp** for LLM inference

### Tiiny AI (NOVA)
- Primary: **PowerInfer** (sparse activation engine)
- Secondary: ONNX Runtime with ARM provider

### Cloud (ECHO)
- Primary: **API proxy** to Shadow Lab inference endpoint
- Secondary: Direct model serving via FastAPI

## Shadow Score Integration

nnn-cli feeds directly into the Shadow Score formula:
```
S = 0.25×ANE_TOPS + 0.25×RAM/5 + 0.20×BW/10 + 0.15×CPU + 0.15×GPU/4
```

`nnn device bench` outputs the actual measured TOPS for the current device, which validates/updates the Shadow Score.

## Example Workflow

```bash
# 1. Check what's available
nnn device list
# ANE: Apple Neural Engine 16-core (38 TOPS)
# GPU: Apple M4 10-core (4.3 TFLOPS)
# CPU: Apple M4 10-core (4P+6E)

# 2. Benchmark ANE
nnn device bench --device npu
# matmul (1024x1024): 0.3ms (ANE)
# conv2d (224x224): 0.8ms (ANE)
# softmax (1x1000): 0.1ms (ANE)

# 3. Run a model on ANE
nnn run nomic-embed.mlx --input "Shadow Lab makes AI coherent" --device npu
# [0.023, -0.118, 0.445, ...] (768-dim embedding in 2ms)

# 4. Route across mesh
nnn mesh route llama3-8b.gguf --input "explain context coherence" --device auto
# Routing to JARVIS (38 TOPS ANE, 16GB) → MLX backend
# Output: "Context coherence is..."
```
