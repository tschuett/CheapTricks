# GlobalIsel

# miniGhost

```shell
> git clone https://github.com/Mantevo/miniGhost.git
> cd ref
> flang-new -D_MG_INT4 -D_MG_REAL8 -ffree-form MG_FLUX_ACCUMULATE.F -emit-llvm -S -o flux_accumulate.ll -O3
> llc -mtriple=aarch64 -global-isel -stop-after=aarch64-prelegalizer-combiner flux_accumulate.ll
```

`flux_accumulate.s` will contain MIR from Fortran code.


# Clang bootstrap

```shell
> cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="AArch64" -DLLVM_USE_LINKER=lld -DLLVM_ENABLE_PROJECTS="clang" -DCLANG_BOOTSTRAP_PASSTHROUGH="LLVM_TARGETS_TO_BUILD" -DCLANG_ENABLE_BOOTSTRAP=On -DBOOTSTRAP_CMAKE_CXX_FLAGS="-march=native -Rpass-missed='gisel*' -fglobal-isel -mllvm -global-isel-abort=2" ./llvm
> ninja stage2
```

Clang will output remarks about failures, i.e.:

```shell
remark: unable to legalize instruction: %151:_(<4 x s64>) = G_CTPOP %149:_(<4 x s64>) [-Rpass-missed=gisel-legalize]
```

# Tablegen import

```shell
 ./bin/llvm-tblgen -gen-global-isel -warn-on-skipped-patterns -I llvm/lib/Target/AArch64 -I include -Illvm/include -Illvm/lib/Target llvm/lib/Target/AArch64/AArch64.td -o AArch64.inc
```

```
./bin/llvm-tblgen -gen-global-isel -warn-on-skipped-patterns -stats -I llvm/lib/Target/AMDGPU -I include -I llvm/include -I llvm/lib/Target llvm/lib/Target/AMDGPU/AMDGPUGISel.td -o AMDGPUGenGlobalISel.inc  2> result-amdgpu.txt
```

```
./bin/llvm-tblgen -gen-global-isel -warn-on-skipped-patterns -stats -I llvm/lib/Target/AArch64 -I include -Illvm/include -Illvm/lib/Target llvm/lib/Target/AArch64/AArch64.td -o AArch64.inc 2> result.txt
```

```
./bin/llvm-tblgen -gen-global-isel  -warn-on-skipped-patterns -stats -I llvm/lib/Target/RISCV -I include -Illvm/include -Illvm/lib/Target llvm/lib/Target/RISCV/RISCVGISel.td -o RISCVGenGlobalISel.inc 2> result-riscv.txt
```

# Sqlite

```
clang --target=arm64-apple-darwin24.4.0 -mcpu=apple-m4 -S ../sqlite-amalgamation-3500400/sqlite3.c -O3 -emit-llvm
llc -mtriple=aarch64-linux-gnu --global-isel -stop-after="aarch64-prelegalizer-combiner" sqlite3.ll
```

# Sema
```
clang --target=arm64-apple-darwin24.4.0 -mcpu=apple-m4 -std=c++17 -Itools/clang/include -Iinclude -Illvm/include -Iclang/include -S clang/lib/Sema/Sema.cpp -O3 -emit-llvm
remove "probe-stack"="__chkstk_darwin"
llc -mtriple=aarch64-linux-gnu --global-isel -stop-after="aarch64-prelegalizer-combiner" Sema.ll
```

