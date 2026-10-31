TAMARIN ?= tamarin-prover
FLAGS   ?= --derivcheck-timeout=300 --open-chains=30
TIMEOUT ?= 3600
export LC_ALL := C.UTF-8

all: $(patsubst %.spthy,results/%.txt,$(wildcard *.spthy))
pcs: $(patsubst %.spthy,results/%.pcs.txt,$(filter-out m1.spthy,$(shell grep -l '^lemma pcs_' *.spthy)))

define prove
@mkdir -p results logs; : > $2.tmp
@for l in $$(sed -En 's/^lemma ([A-Za-z0-9_]+).*/\1/p' $1 | grep $3); do \
  log=logs/$(basename $1).$$l.log; \
  timeout $(TIMEOUT) $(TAMARIN) $(FLAGS) --prove=$$l $1 > $$log 2>&1; \
  test -s $2.tmp || grep -m1 -o 'WARNING: .*wellformedness.*' $$log >> $2.tmp; \
  sed -En "s/^  ($$l \((all-traces|exists-trace)\): .*) \([0-9]+ steps\)\$$/\1/p" $$log \
    | grep . >> $2.tmp || echo "$$l: no result (timeout or error)" >> $2.tmp; \
done; mv $2.tmp $2
@echo "== $1"; cat $2
endef

results/%.txt: %.spthy
	$(call prove,$<,$@,$(if $(filter m1,$*),.,-v '^pcs_'))

results/%.pcs.txt: %.spthy
	$(call prove,$<,$@,'^pcs_')

lemma:
	@test -n "$(T)" -a -n "$(L)" || { echo "usage: make lemma T=<theory> L=<lemma>"; exit 2; }
	@mkdir -p logs; timeout $(TIMEOUT) $(TAMARIN) $(FLAGS) --prove=$(L) $(T).spthy > logs/$(T).$(L).log 2>&1; \
	grep -E '^  $(L) \((all-traces|exists-trace)\)' logs/$(T).$(L).log || echo "$(L): no result (timeout or error)"

check: all
	@test -z "$$(git status --porcelain -- results)" || \
	  { git --no-pager diff -- results; git status --short -- results; exit 1; }

.PHONY: all check pcs lemma
