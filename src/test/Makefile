PYTHON ?= python
CPPFLAGS += -I.. $(shell $(PYTHON) -c "from __future__ import print_function; import sys; print('-DPYCRYPTO_' + sys.byteorder.upper() + '_ENDIAN')")
CPPFLAGS += -DHAVE_POSIX_MEMALIGN -DHAVE_STDINT_H
CPPFLAGS += -DSTATIC=""
CFLAGS += -O3 -g -fstrict-aliasing -Wall -Werror

TAPPS:=tests_addmul128_32 tests_addmul128_64 tests_square_32 tests_square_64 tests_product tests_addmul test_endianess\
       test_poly1305_reduce test_poly1305_load_r test_poly1305_load_m test_poly1305_multiply test_poly1305_accumulate\
       test_mont tests_mont_mult tests_ec_ws_64 tests_ec_ws_32 test_pkcs1

ifneq (,$(filter $(shell uname -m),x86_64 i386 i686))
CPPFLAGS += -DHAVE_X86INTRIN_H -DUSE_SSE2 -DHAVE_WMMINTRIN_H -DHAVE_TMMINTRIN_H
CFLAGS += -msse2
TAPPS += test_clmul
endif

ifeq (64bit, $(shell $(PYTHON) -c "from __future__ import print_function; import platform; print(platform.architecture()[0])"))
CPPFLAGS += -DHAVE_UINT128
endif

CPPFLAGS += $(UNDEFS)

TGTS:=$(foreach t,$(TAPPS),build/$t)

UTILS=build/modexp_utils.o build/siphash.o

test: build ${TGTS}
	set -e && for t in ${TGTS}; do $$t; done

build:
	mkdir build

all: ${$TGTS}

clean:
	rm -fr build common.pyc

# utils

build/siphash.o: ../siphash.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $^

build/modexp_utils.o: ../modexp_utils.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $^

# ECC

TABLES = build/p256_table.o build/p384_table.o build/p521_table.o

build/p256_table.o: ../p256_table.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -c -o $@ $^

build/p384_table.o: ../p384_table.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -c -o $@ $^

build/p521_table.o: ../p521_table.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -c -o $@ $^

build/tests_ec_ws_64: test_ec_ws.c ../ec_ws.c build/mont_64.o $(TABLES) $(UTILS)
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

build/tests_ec_ws_32: test_ec_ws.c ../ec_ws.c build/mont_32.o $(TABLES) $(UTILS)
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^ -DSYS_BITS=32 -UHAVE_UINT128

# addmul128

build/tests_addmul128.c: make_tests_addmul128.py
	$(PYTHON) $^ > $@

build/tests_addmul128_32: build/tests_addmul128.c ../multiply_32.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

build/tests_addmul128_64: build/tests_addmul128.c ../multiply_64.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

# square

build/tests_square.c: make_tests_square.py
	$(PYTHON) $^ > $@

build/tests_square_32: build/tests_square.c ../multiply_32.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

build/tests_square_64: build/tests_square.c ../multiply_64.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

# endianess

build/test_endianess: test_endianess.c ../common.h
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $<

# clmul

build/clmul.o: ../ghash_clmul.c
	$(CC) $(CFLAGS) -mssse3 -mpclmul $(CPPFLAGS) -o $@ $< -c

build/test_clmul: test_clmul.c ../common.h build/clmul.o
	$(CC) $(CFLAGS) -mssse3 -mpclmul $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

# Poly1305

build/poly1305.o: ../poly1305.c
	$(CC) $(CFLAGS) $(CPPFLAGS) -o $@ $^ -c

build/test_poly1305_reduce.c: make_tests_poly1305_reduce.py
	$(PYTHON) $^ > $@

build/test_poly1305_load_r.c: make_tests_poly1305_load_r.py
	$(PYTHON) $^ > $@

build/test_poly1305_load_m.c: make_tests_poly1305_load_m.py
	$(PYTHON) $^ > $@

build/test_poly1305_multiply.c: make_tests_poly1305_multiply.py
	$(PYTHON) $^ > $@

build/test_poly1305_accumulate.c: make_tests_poly1305_accumulate.py
	$(PYTHON) $^ > $@

build/test_poly1305_reduce: build/test_poly1305_reduce.c ../common.h build/poly1305.o
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

build/test_poly1305_load_r: build/test_poly1305_load_r.c ../common.h build/poly1305.o
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

build/test_poly1305_load_m: build/test_poly1305_load_m.c ../common.h build/poly1305.o
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

build/test_poly1305_multiply: build/test_poly1305_multiply.c ../common.h build/poly1305.o
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

build/test_poly1305_accumulate: build/test_poly1305_accumulate.c ../common.h build/poly1305.o
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

# Montgomery

build/mont_32.o: ../mont.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^ -DSYS_BITS=32 -UHAVE_UINT128

build/mont_64.o: ../mont.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^ -DSYS_BITS=64

build/test_mont: test_mont.c ../common.h build/mont_32.o $(UTILS)
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $(filter %.c %.o, $^)

build/tests_addmul.c: make_tests_addmul.py
	$(PYTHON) $^ > $@

build/tests_addmul: build/tests_addmul.c build/mont_32.o $(UTILS)
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

build/tests_product.c: make_tests_product.py
	$(PYTHON) $^ > $@

build/tests_product: build/tests_product.c build/mont_32.o $(UTILS)
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

build/tests_mont_mult.c: make_tests_mont_mult.py
	$(PYTHON) $^ > $@

build/tests_mont_mult: build/tests_mont_mult.c build/mont_32.o $(UTILS)
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^

# pkcs1
build/test_pkcs1: test_pkcs1.c ../common.h ../pkcs1_decode.c
	$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ $^
