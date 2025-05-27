# MustAI - Local LLaMA Chat Interface

Bu proje, yerel olarak LLaMA modellerini çalıştırmak için Go tabanlı bir chat arayüzüdür.

## Özellikler

- Local LLaMA model desteği (GGUF formatı)
- Interaktif chat arayüzü
- GPU acceleration desteği
- Multi-threading support
- Embedding generation

## Kurulum

### Gereksinimler

- Go 1.19+
- GCC/G++ compiler
- Git

### Linux/WSL Kurulumu

```bash
# Gerekli paketleri kurun
sudo apt update
sudo apt install -y build-essential cmake git

# Repository'yi klonlayın
git clone --recurse-submodules https://github.com/go-skynet/go-llama.cpp

# Backend dizinine gidin
cd backend

# LLaMA.cpp'yi build edin
cd llama.cpp
make clean
make

# Static library oluşturun
ar rcs libllama.a ggml.o llama.o common.o k_quants.o ggml-alloc.o

# Ana dizine dönün
cd ..
```

## Model İndirme

Hugging Face'den GGUF formatında model indirin:

```bash
# Models dizinini oluşturun
mkdir -p llama.cpp/models

# LLaMA-2-7B-Chat modelini indirin
wget -O llama.cpp/models/llama-2-7b-chat.Q2_K.gguf \
  "https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/resolve/main/llama-2-7b-chat.Q2_K.gguf"
```

## Kullanım

### Temel Kullanım

```bash
# Environment değişkenlerini ayarlayın
export CGO_CFLAGS="-I./llama.cpp"
export CGO_LDFLAGS="-L./llama.cpp -lllama -lm -lstdc++"

# Uygulamayı çalıştırın
go run ./cmd -t 14
```

### Parametreler

- `-m`: Model dosyası yolu
- `-t`: Thread sayısı
- `-n`: Token sayısı
- `-ngl`: GPU layer sayısı
- `-s`: Random seed

### Örnek Kullanım

```bash
# Özel model ile çalıştırma
go run ./cmd -m "./llama.cpp/models/custom-model.gguf" -t 8 -n 256

# GPU acceleration ile
go run ./cmd -ngl 32 -t 16
```

## GPU Acceleration

### CUDA (NVIDIA)

```bash
BUILD_TYPE=cublas make libbinding.a
CGO_LDFLAGS="-lcublas -lcudart -L/usr/local/cuda/lib64/" \
LIBRARY_PATH=$PWD C_INCLUDE_PATH=$PWD go run ./cmd -ngl 32
```

### OpenBLAS

```bash
BUILD_TYPE=openblas make libbinding.a
CGO_LDFLAGS="-lopenblas" \
LIBRARY_PATH=$PWD C_INCLUDE_PATH=$PWD go run ./cmd
```

## Troubleshooting

### Linkage Hataları

Eğer `-lbinding` veya `-lllama` hataları alıyorsanız:

```bash
# Static library'yi yeniden oluşturun
cd llama.cpp
make clean
make
ar rcs libllama.a *.o

# Cache'i temizleyin
cd ..
go clean -cache
```

### Model Yükleme Hataları

- Model dosyasının GGUF formatında olduğundan emin olun
- Dosya boyutunu ve bütünlüğünü kontrol edin
- Yeterli RAM olduğundan emin olun

### Compilation Warnings

C++ binding uyarıları normal ve güvenlidir:

```bash
binding.cpp:809:5: warning: possible problem detected in invocation of 'operator delete'
```

Bu uyarıyı bastırmak için:

```bash
export CGO_CFLAGS="-I./llama.cpp -Wno-delete-incomplete"
```

### Performance Optimizasyonu

- CPU-bound işlemler için thread sayısını artırın: `-t 16`
- GPU kullanıyorsanız layer sayısını ayarlayın: `-ngl 32`
- Küçük modeller daha hızlı çalışır (Q2_K < Q4_0 < Q8_0)
- Context boyutunu ihtiyacınıza göre ayarlayın

### WSL Specific Notes

Windows WSL kullanıyorsanız:

```bash
# WSL path kullanın
cd /mnt/d/APP/mustAI/backend

# Windows'tan dosya erişimi için
export WSLENV=PATH/l
```

## Desteklenen Model Formatları

- GGUF (önerilen)
- Quantized models (Q2_K, Q4_0, Q8_0, vs.)

## Katkıda Bulunma

1. Fork yapın
2. Feature branch oluşturun
3. Değişikliklerinizi commit edin
4. Pull request gönderin

## Lisans

MIT License
