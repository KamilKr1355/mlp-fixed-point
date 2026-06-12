# Optymalizacja sieci MLP oraz CNN : FP32 vs Fixed-Point (PTQ i QAT)

Projekt badawczy demonstrujący wpływ kwantyzacji sieci neuronowej (Multi-Layer Perceptron oraz CNN) na jej rozmiar, czas inferencji oraz dokładność klasyfikacji na zbiorze MNIST oraz cifar10. W projekcie porównujemy trzy podejścia:
1. **Model bazowy (Float32)** - standardowa precyzja zmiennoprzecinkowa.
2. **PTQ (Post-Training Quantization)** - kwantyzacja do 8-bitowej arytmetyki (INT8) po zakończeniu trenowania.
3. **QAT (Quantization-Aware Training)** - trenowanie modelu ze świadomością kwantyzacji (symulacja błędów obcięcia ułamków), a następnie konwersja do INT8.

Raport z doświadczenia został zapisany w pliku raport.md

## Wymagania

Jeśli nie masz zainstalowanego `uv`, zainstaluj go:
- **Windows (PowerShell):** `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`
- **macOS/Linux:** `curl -LsSf https://astral.sh/uv/install.sh | sh`

## Uruchamianie

   ```bash
   git clone https://github.com/KamilKr1355/mlp-fixed-point
   cd mlp-fixed-point
   uv sync
