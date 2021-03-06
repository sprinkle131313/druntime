include ../common.mak

TESTS:=stderr_msg unittest_assert invalid_memory_operation unknown_gc static_dtor future_message

ifeq ($(OS)-$(BUILD),linux-debug)
	TESTS:=$(TESTS) line_trace rt_trap_exceptions
endif
ifeq ($(OS)-$(BUILD),freebsd-debug)
	TESTS:=$(TESTS) line_trace
endif

DIFF:=diff
SED:=sed

.PHONY: all clean
all: $(addprefix $(ROOT)/,$(addsuffix .done,$(TESTS)))

$(ROOT)/line_trace.done: $(ROOT)/line_trace
	@echo Testing line_trace
	$(QUIET)$(TIMELIMIT)$(ROOT)/line_trace $(RUN_ARGS) > $(ROOT)/line_trace.output
	# Use sed to canonicalize line_trace.output and compare against expected output in line_trace.exp
	$(QUIET)$(SED) "s/\[0x[0-9a-f]*\]/\[ADDR\]/g; s/scope //g; s/Nl//g" $(ROOT)/line_trace.output | $(DIFF) line_trace.exp -
	@rm -f $(ROOT)/line_trace.output
	@touch $@

$(ROOT)/chain.done: $(ROOT)/chain
	@echo Testing chain
	$(QUIET)$(TIMELIMIT)$(ROOT)/chain $(RUN_ARGS) > $(ROOT)/chain.output
	@rm -f $(ROOT)/chain.output
	@touch $@

$(ROOT)/stderr_msg.done: STDERR_EXP="stderr_msg msg"
$(ROOT)/unittest_assert.done: STDERR_EXP="unittest_assert msg"
$(ROOT)/invalid_memory_operation.done: STDERR_EXP="InvalidMemoryOperationError"
$(ROOT)/unknown_gc.done: STDERR_EXP="'unknowngc'"
$(ROOT)/static_dtor.done: STDERR_EXP="dtor_called_more_than_once"
$(ROOT)/future_message.done: STDERR_EXP="exception I have a custom message. exception exception "
$(ROOT)/static_dtor.done: NEGATE=!
$(ROOT)/rt_trap_exceptions.done: STDERR_EXP="uncaught exception\nobject.Exception@rt_trap_exceptions.d(11): exception"
$(ROOT)/rt_trap_exceptions.done: NEGATE=!
$(ROOT)/%.done: $(ROOT)/%
	@echo Testing $*
	$(NEGATE) $(QUIET)$(TIMELIMIT)$(ROOT)/$* $(RUN_ARGS) 2>&1 1>/dev/null | grep -qF $(STDERR_EXP)
	@touch $@

$(ROOT)/unittest_assert: DFLAGS+=-unittest
$(ROOT)/line_trace: DFLAGS+=-L--export-dynamic
$(ROOT)/%: $(SRC)/%.d
	$(QUIET)$(DMD) $(DFLAGS) -of$@ $<

clean:
	rm -rf $(GENERATED)
