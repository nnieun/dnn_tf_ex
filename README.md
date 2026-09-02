






cat > run_gpu_code.sh <<'EOF'
#!/bin/bash

export LD_LIBRARY_PATH=$(find .venv/lib/python*/site-packages/nvidia \
  -type d -name lib | tr '\n' ':'):$LD_LIBRARY_PATH

uv run "$@"
EOF

chmod +x run_gpu_code.sh

./run_gpu_code.sh python -c "import tensorflow as tf; print('TensorFlow:', tf.__version__); print('GPU:', tf.config.list_physical_devices('GPU'))"

