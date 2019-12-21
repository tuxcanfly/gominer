CC ?= gcc -fPIC
CXX ?= g++ -fPIC
NVCC ?= nvcc -Xcompiler -fPIC
AR ?= ar
# -o is gnu only so this needs to be smarter; it does work because on darwin it
#  fails which is also not windows.
ARCH:=$(shell uname -o)

.DEFAULT_GOAL := build

ifeq ($(ARCH),Msys)
nvidia:
endif

# Windows needs additional setup and since cgo does not support spaces in
# in include and library paths we copy it to the correct location.
#
# Windows build assumes that CUDA V7.0 is installed in its default location.
#
# Windows gominer requires nvml.dll and handshake.dll to reside in the same
# directory as gominer.exe.
ifeq ($(ARCH),Msys)
obj: nvidia
	mkdir nvidia
	cp -r /c/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/* nvidia
	cp -r /c/Program\ Files/NVIDIA\ Corporation/NVSMI nvidia
else
obj:
endif
	mkdir obj

ifeq ($(ARCH),Msys)
obj/handshake.dll: obj sph/blake.c handshake.cu
	$(NVCC) --shared --optimize=3 --compiler-options=-GS-,-MD -I. -Isph handshake.cu sph/blake.c -o obj/handshake.dll
else
obj/libhandshake.so: obj sph/blake.c handshake.cu
	$(NVCC) --shared --optimize=3 -I. handshake.cu sph/blake.c -o obj/libhandshake.so
endif

ifeq ($(ARCH),Msys)
build: obj/handshake.dll
else
build: obj/libhandshake.so
endif
	go build -tags 'cuda'

ifeq ($(ARCH),Msys)
install: obj/handshake.dll
else
install: obj/libhandshake.so
endif
	go install -tags 'cuda'

clean:
	rm -rf obj
	go clean
ifeq ($(ARCH),Msys)
	rm -rf nvidia
endif
