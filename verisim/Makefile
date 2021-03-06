base_dir=$(abspath ..)
sim_dir=$(abspath .)

include $(base_dir)/Makefrag-variables
sim = $(sim_dir)/simulator-$(PROJECT)-$(CONFIG)
sim_debug = $(sim_dir)/simulator-$(PROJECT)-$(CONFIG)-debug
tether_file = $(build_dir)/$(CONFIG).tether
TETHER ?= dtm

default: $(sim)

debug: $(sim_debug)

CXXFLAGS := $(CXXFLAGS) -O1 -std=c++11 -I$(RISCV)/include -D__STDC_FORMAT_MACROS
LDFLAGS := $(LDFLAGS) -L$(RISCV)/lib -Wl,-rpath,$(RISCV)/lib -L$(sim_dir) -lfesvr -lpthread

include $(base_dir)/Makefrag
include $(sim_dir)/Makefrag-verilator
ifneq ($(filter run% %.run %.out %.vpd %.vcd,$(MAKECMDGOALS)),)
-include $(build_dir)/$(long_name).d
endif

sim_csrcs += \
	$(sim_dir)/csrc/verilator-harness.cc

model_dir = $(build_dir)/$(long_name)
model_dir_debug = $(build_dir)/$(long_name).debug

model_header = $(model_dir)/V$(MODEL).h
model_header_debug = $(model_dir_debug)/V$(MODEL).h

model_mk = $(model_dir)/V$(MODEL).mk
model_mk_debug = $(model_dir_debug)/V$(MODEL).mk

$(model_mk): $(sim_vsrcs) $(INSTALLED_VERILATOR)
	rm -rf $(build_dir)/$(long_name)
	mkdir -p $(build_dir)/$(long_name)
	$(if $(wildcard $(tether_file)),$(eval TETHER := tsi))
	$(VERILATOR) $(VERILATOR_FLAGS) -Mdir $(build_dir)/$(long_name) \
	-o $(sim) $< $(sim_csrcs) -LDFLAGS "$(LDFLAGS)" \
	-CFLAGS "-I$(build_dir) -DTETHER_TYPE=$(TETHER)_t -DTETHER_NAME=$(TETHER) -include $(model_header)"
	touch $@

$(sim): $(model_mk) $(sim_csrcs)
	$(MAKE) VM_PARALLEL_BUILDS=1 -C $(build_dir)/$(long_name) -f V$(MODEL).mk


$(model_mk_debug): $(sim_vsrcs) $(INSTALLED_VERILATOR)
	mkdir -p $(build_dir)/$(long_name).debug
	$(if $(wildcard $(tether_file)),$(eval TETHER := tsi))
	$(VERILATOR) $(VERILATOR_FLAGS) -Mdir $(build_dir)/$(long_name).debug --trace \
	-o $(sim_debug) $< $(sim_csrcs) -LDFLAGS "$(LDFLAGS)" \
	-CFLAGS "-I$(build_dir) -DTETHER_TYPE=$(TETHER)_t -DTETHER_NAME=$(TETHER) -include $(model_header_debug)"
	touch $@

$(sim_debug): $(model_mk_debug) $(sim_csrcs)
	$(MAKE) VM_PARALLEL_BUILDS=1 -C $(build_dir)/$(long_name).debug -f V$(MODEL).mk


exec_sim = $(sim)
exec_sim_debug = $(sim_debug)

clean:
	rm -rf $(build_dir) ./simulator-*

.PHONY: clean default debug
