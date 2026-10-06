# Run every Tamarin theory in this directory and record which lemmas hold.
#
#   make          prove theories that changed -> results/<name>.txt, one line per lemma
#   make -B       re-prove everything
#   make check    as above, then fail if any result differs from what is committed
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

# One Tamarin run per lemma. A lemma that does not terminate costs one TIMEOUT
# and shows as "no result"; the others still report. Memory is bounded by the
# largest single proof. The price is one source precomputation per lemma.
# Full output of each run goes to logs/<theory>.<lemma>.log.
results/%.txt: %.spthy
	@mkdir -p results logs; : > $@.tmp
	@for l in $$(sed -En 's/^lemma ([A-Za-z0-9_]+).*/\1/p' $<); do \
	  log=logs/$*.$$l.log; \
	  timeout $(TIMEOUT) $(TAMARIN) $(FLAGS) --prove=$$l $< > $$log 2>&1; \
	  test -s $@.tmp || grep -m1 -o 'WARNING: .*wellformedness.*' $$log >> $@.tmp; \
	  sed -En "s/^  ($$l \((all-traces|exists-trace)\): .*) \([0-9]+ steps\)\$$/\1/p" $$log \
	    | grep . >> $@.tmp || echo "$$l: no result (timeout or error)" >> $@.tmp; \
	done; mv $@.tmp $@
	@echo "== $<"; cat $@

check: all
	@test -z "$$(git status --porcelain -- results)" || \
	  { git --no-pager diff -- results; git status --short -- results; exit 1; }

.PHONY: all check
