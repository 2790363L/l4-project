# Run every Tamarin theory in this directory and record which lemmas hold.
#
#   make          prove theories that changed -> results/<name>.txt, one line per lemma
#   make -B       re-prove everything
#   make check    as above, then fail if any result differs from what is committed
#   make pcs      the pcs_* lemmas of M2 onward -> results/<name>.pcs.txt
#   make lemma T=m2a L=tree_invariant_X
#                 one lemma, same flags; full output in logs/m2a.tree_invariant_X.log
#
# results/ is committed, so accepting a changed result = committing it.
# Full Tamarin output (step counts, timings) goes to logs/, which is not committed.

TAMARIN ?= tamarin-prover
# The M1 rules are too large for the default 5-second derivation check.
# m1_remove.spthy needs more than the default 10 open chains, or Tamarin
# stops its source precomputation early.
FLAGS   ?= --derivcheck-timeout=300 --open-chains=30
# Seconds per lemma.
TIMEOUT ?= 3600
export LC_ALL := C.UTF-8

all: $(patsubst %.spthy,results/%.txt,$(wildcard *.spthy))

# Whether PCS stays in scope is open with Shahid, so pcs_* lemmas are left out
# of `make` and run by `make pcs`. m1.spthy is the exception: pcs_epoch2 is
# already in results/m1.txt and stays there.
pcs: $(patsubst %.spthy,results/%.pcs.txt,$(filter-out m1.spthy,$(shell grep -l '^lemma pcs_' *.spthy)))

# $(call prove,THEORY,RESULTS,GREP-ARGS): one Tamarin run per lemma whose name
# passes `grep GREP-ARGS`. A lemma that does not terminate costs one TIMEOUT
# and shows as "no result"; the others still report. Memory is bounded by the
# largest single proof. The price is one source precomputation per lemma.
# Full output of each run goes to logs/<theory>.<lemma>.log.
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
