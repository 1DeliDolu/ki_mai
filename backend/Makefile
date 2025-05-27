.PHONY: build clean

build: llama.cpp/libllama.a
    CGO_CFLAGS="-I./llama.cpp" \
    CGO_LDFLAGS="-L./llama.cpp -lllama -lm -lstdc++" \
    go build -o bin/mustai ./cmd

llama.cpp/libllama.a:
    cd llama.cpp && make libllama.a

clean:
    cd llama.cpp && make clean
    rm -f bin/mustai

run: build
    ./bin/mustai -t 14
