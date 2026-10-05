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
FLAGS   ?= --derivcheck-timeout=120
TIMEOUT ?= 3600
export LC_ALL := C.UTF-8

all: $(patsubst %.spthy,results/%.txt,$(wildcard *.spthy))

# ponytail: one timeout per file, so a single non-terminating lemma blanks the
# whole theory. Switch to per-lemma runs (--prove=NAME) when that starts to hurt.
results/%.txt: %.spthy
	@mkdir -p results logs
	@timeout $(TIMEOUT) $(TAMARIN) $(FLAGS) --prove $< > logs/$*.log 2>&1 || true
	@{ grep -m1 -o 'WARNING: .*wellformedness.*' logs/$*.log; \
	   sed -En 's/^  (\S+ \((all-traces|exists-trace)\): .*) \([0-9]+ steps\)$$/\1/p' logs/$*.log; } > $@
	@test -s $@ || echo 'no result (timeout or error)' > $@
	@echo "== $<"; cat $@

check: all
	@test -z "$$(git status --porcelain -- results)" || \
	  { git --no-pager diff -- results; git status --short -- results; exit 1; }

.PHONY: all check
