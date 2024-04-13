grain compile adder/adder.gr --release  -o adder.wasm
grain compile calculator/calculator.gr --release  -o calculator.wasm
grain compile command/command.gr --release --use-start-section -o command.wasm


wasm-tools component embed ./wit --world adder -o embedded-adder.wasm adder.wasm

wasm-tools component embed ./wit --world calculator -o embedded-calculator.wasm calculator.wasm

wasm-tools component new -o component-adder.wasm --adapt ./adapters/wasi_snapshot_preview1.command.wasm embedded-adder.wasm

wasm-tools component new -o component-calculator.wasm --adapt ./adapters/wasi_snapshot_preview1.command.wasm embedded-calculator.wasm

wasm-tools compose component-calculator.wasm -d component-adder.wasm -o composed-calculator.wasm

wasm-tools component embed ./wit --world app -o command.embedded.wasm command.wasm

wasm-tools component new -o component-command.wasm --adapt ./adapters/wasi_snapshot_preview1.command.wasm command.embedded.wasm

wasm-tools compose component-command.wasm -d composed-calculator.wasm -o final.wasm

wasmtime -W tail-call final.wasm