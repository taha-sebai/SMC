# ============================================================
# Makefile - TP1 SoCLib
# ============================================================

CXX = g++

# Options communes
CXXFLAGS = -Wno-deprecated -fpermissive -std=gnu++0x -m32

# Chemins SystemC classique
SYSTEMC_INCLUDE = /users/outil/dsx/cctools/include
SYSTEMC_LIB     = /users/outil/dsx/cctools/lib-linux

# Chemins SystemCASS
SYSTEMCASS_INCLUDE = /users/outil/dsx/systemcass/include
SYSTEMCASS_LIB     = /users/outil/dsx/systemcass/lib


# ============================================================
# Cibles principales
# ============================================================

.PHONY: all clean run fast-run

all: simulator.x fast_simulator.x


# ============================================================
# SystemC classique
# ============================================================

simulator.x: fifo_gcd_master.o fifo_gcd_coprocessor.o tp1_top.o
	$(CXX) $(CXXFLAGS) \
		-L. \
		-L$(SYSTEMC_LIB) \
		-o $@ \
		$^ \
		-lsystemc \
		2>&1


fifo_gcd_master.o: fifo_gcd_master.cpp fifo_gcd_master.h fifo_ports.h fifo_signals.h
	$(CXX) $(CXXFLAGS) \
		-I. \
		-I$(SYSTEMC_INCLUDE) \
		-c $< \
		-o $@


fifo_gcd_coprocessor.o: fifo_gcd_coprocessor.cpp fifo_gcd_coprocessor.h fifo_ports.h fifo_signals.h
	$(CXX) $(CXXFLAGS) \
		-I. \
		-I$(SYSTEMC_INCLUDE) \
		-c $< \
		-o $@


tp1_top.o: tp1_top.cpp fifo_gcd_master.h fifo_gcd_coprocessor.h fifo_ports.h fifo_signals.h
	$(CXX) $(CXXFLAGS) \
		-I. \
		-I$(SYSTEMC_INCLUDE) \
		-c $< \
		-o $@


# ============================================================
# SystemCASS
# ============================================================

fast_simulator.x: fifo_gcd_master_fast.o fifo_gcd_coprocessor_fast.o tp1_top_fast.o
	$(CXX) \
		-Wno-deprecated \
		-fpermissive \
		-m32 \
		-L. \
		-L$(SYSTEMCASS_LIB) \
		-o $@ \
		$^ \
		-lsystemc \
		-ldl \
		2>&1


fifo_gcd_master_fast.o: fifo_gcd_master.cpp fifo_gcd_master.h fifo_ports.h fifo_signals.h
	$(CXX) \
		-Wno-deprecated \
		-fpermissive \
		-I. \
		-I$(SYSTEMCASS_INCLUDE) \
		-m32 \
		-c $< \
		-o $@


fifo_gcd_coprocessor_fast.o: fifo_gcd_coprocessor.cpp fifo_gcd_coprocessor.h fifo_ports.h fifo_signals.h
	$(CXX) \
		-Wno-deprecated \
		-fpermissive \
		-I. \
		-I$(SYSTEMCASS_INCLUDE) \
		-m32 \
		-c $< \
		-o $@


tp1_top_fast.o: tp1_top.cpp fifo_gcd_master.h fifo_gcd_coprocessor.h fifo_ports.h fifo_signals.h
	$(CXX) \
		-Wno-deprecated \
		-fpermissive \
		-I. \
		-I$(SYSTEMCASS_INCLUDE) \
		-m32 \
		-c $< \
		-o $@


# ============================================================
# Exécution
# ============================================================

run: simulator.x
	./simulator.x 10000

fast-run: fast_simulator.x
	./fast_simulator.x 10000


# ============================================================
# Nettoyage
# ============================================================

clean:
	rm -f *.o simulator.x fast_simulator.x