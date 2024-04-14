
This is my workings to recreate the Bytecode alliance WebAssembly Component Model tutorial (https://component-model.bytecodealliance.org/tutorial.html) but in Grain instead of rust.

Ue the Grain branch of wit-bindgen (https://github.com/grain-lang/wit-bindgen/tree/grain) to generate the skeleton code

```wit-bindgen grain <witfile> -w <world>```

So for example wit-bindgen grain calculator.wit -w adder

I then copied these files into their source locations and added the required functions

To perform the basic compilation

```grain compile adder/adder.gr --release  -o adder.wasm```

```grain compile calculator/calculator.gr --release  -o calculator.wasm```

and for the one I would actually run as a command line program:

```grain compile command/command.gr --release --use-start-section -o command.wasm```


We then use wasm-tools to embed the wit into the wasm

```wasm-tools component embed ./wit --world adder -o embedded-adder.wasm adder.wasm```

```wasm-tools component embed ./wit --world calculator -o embedded-calculator.wasm calculator.wasm```

```wasm-tools component embed ./wit --world app -o command.embedded.wasm command.wasm```


And then use wasm-tooms to make the components:

Make the adder component and add the adder component 

```wasm-tools component new -o component-adder.wasm --adapt ./adapters/wasi_snapshot_preview1.command.wasm embedded-adder.wasm```

Make the calculator component 

```wasm-tools component new -o component-calculator.wasm --adapt ./adapters/wasi_snapshot_preview1.command.wasm embedded-calculator.wasm```

Make the command line compponent

```wasm-tools component new -o component-command.wasm --adapt ./adapters/wasi_snapshot_preview1.command.wasm command.embedded.wasm```

Compose a calculator component with the adder component

```wasm-tools compose component-calculator.wasm -d component-adder.wasm -o composed-calculator.wasm```

Compose the final program with the command component and the composed calculator component

```wasm-tools compose component-command.wasm -d composed-calculator.wasm -o final.wasm```

And finally run it

```wasmtime -W tail-call final.wasm```
