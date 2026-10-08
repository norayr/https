DEPEND = github.com/norayr/strutils github.com/norayr/base64 github.com/norayr/Internet github.com/norayr/http github.com/norayr/tls

VOC = voc
mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
mkfile_dir_path := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
$(info $$mkfile_path is [${mkfile_path}])
$(info $$mkfile_dir_path is [${mkfile_dir_path}])
ifndef BUILD
BUILD="build"
endif
build_dir_path := $(mkfile_dir_path)/$(BUILD)
current_dir := $(notdir $(patsubst %/,%,$(dir $(mkfile_path))))
BLD := $(mkfile_dir_path)/build
DPD  =  deps
ifndef DPS
DPS := $(mkfile_dir_path)/$(DPD)
endif
# the test servers of the tls repository, and how long they run
SERVERS = $(DPS)/github.com/norayr/tls/tools/testservers.py
SECS = 120
all: get_deps build_deps buildThis

get_deps:
	@for i in $(DEPEND); do \
			if [ -d "$(DPS)/$${i}" ]; then \
				 cd "$(DPS)/$${i}"; \
				 git pull; \
				 cd - ;    \
				 else \
				 mkdir -p "$(DPS)/$${i}"; \
				 cd "$(DPS)/$${i}"; \
				 cd .. ; \
				 git clone "https://$${i}"; \
				 cd - ; \
			fi; \
	done

build_deps:
	mkdir -p $(BLD)
	cd $(BLD); \
	for i in $(DEPEND); do \
		if [ -f "$(DPS)/$${i}/GNUmakefile" ]; then \
			make -f "$(DPS)/$${i}/GNUmakefile" BUILD=$(BLD); \
		else \
			make -f "$(DPS)/$${i}/Makefile" BUILD=$(BLD); \
		fi; \
	done

buildThis:
	cd $(BUILD) && $(VOC) -s $(mkfile_dir_path)/src/https.Mod
	cd $(BUILD) && $(VOC) -m $(mkfile_dir_path)/src/fetch.Mod

# the TLS test servers (python3 with cryptography) and a plain HTTP server, both stopped after
# the test; make tests NET=net also downloads from example.com
tests:
	cd $(BUILD) && $(VOC) -m $(mkfile_dir_path)/test/testHttps.Mod
	@d=$$(mktemp -d); \
	python3 $(SERVERS) $(SECS) $$d > $$d/servers.log 2>&1 & sp=$$!; \
	(cd $$d && exec timeout $(SECS) python3 -m http.server 18480 --bind 127.0.0.1 > /dev/null 2>&1) & hp=$$!; \
	n=0; while [ ! -f $$d/big.ref ] && [ $$n -lt 150 ]; do sleep 0.2; n=$$((n + 1)); done; sleep 1; \
	cd $(BUILD) && SSL_CERT_FILE=$$d/ca.pem timeout $(SECS) ./testHttps $$d; r=$$?; \
	kill $$sp $$hp 2>/dev/null; wait 2>/dev/null; rm -rf $$d; \
	if [ $$r = 0 ] && [ -n "$(NET)" ]; then timeout $(SECS) ./testHttps net; r=$$?; fi; exit $$r

clean:
	if [ -d "$(BUILD)" ]; then rm -rf $(BLD); fi
