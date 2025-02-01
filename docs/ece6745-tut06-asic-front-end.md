
ECE 6745 Tutorial 5: ASIC Front-End Flow
==========================================================================

The tutorial will discuss the key tools used for ASIC front-end flow
which includes RTL simulation, synthesis, and fast-functional gate-level
simulation. This tutorial requires entering commands manually for each of
the tools to enable students to gain a better understanding of the
detailed steps involved in this process. A later tutorial will illustrate
how this process can be automated to facilitate rapid design-space
exploration. This tutorial assumes you have already completed the
tutorials on Linux, Git, and Verilog.

The following diagram illustrates the five primary tools we will be using
in ECE 6745 along with a few smaller secondary tools. The tools that
make-up the ASIC front-end flow are highlighted in red. Notice that the
ASIC tools all require various views from the standard-cell library.
Before starting this tutorial, you must complete the ASIC standard-cell
tutorial so you can understand all of these views.

![](img/tut06-asic-flow-front-end.png)

 1. We write our RTL models in Verilog, and we use the PyMTL framework to
    test, verify, and evaluate the execution time (in cycles) of our
    design. This part of the flow is very similar to the flow used in
    ECE 4750. Once we are sure our design is working correctly, we can
    then start to push the design through the flow.

 2. We use **Synopsys VCS** to compile and run both 4-state RTL and
    gate-level simulations. These simulations help us to build confidence
    in our design as we push our designs through different stages of the
    flow. From these simulations, we also generate waveforms in `.vcd`
    (Verilog Change Dump) format, and we use `vcd2saif` to convert these
    waveforms into per-net average activity factors stored in `.saif`
    format. These activity factors will be used for power analysis.
    Gate-level simulation is an valuable tool for ensuring the tools did
    not optimize something away which impacts the correctness of the
    design, and also provides an avenue for obtaining a more accurate
    power analysis than RTL simulation. While static timing analysis
    (STA) analyzes all paths, GL simulation can also serve as a backup to
    check for hold and setup time violations (chip designers must be
    paranoid!)

 3. We use **Synopsys Design Compiler (DC)** to synthesize our design,
    which means to transform the Verilog RTL model into a Verilog
    gate-level netlist where all of the gates are selected from the
    standard-cell library. We need to provide Synopsys DC with abstract
    logical and timing views of the standard-cell library in `.db`
    format. In addition to the Verilog gate-level netlist, Synopsys DC
    can also generate a `.ddc` file which contains information about the
    gate-level netlist and timing, and this `.ddc` file can be inspected
    using Synopsys Design Vision (DV).

 4. We use **Cadence Innovus** to place-and-route our design, which means
    to place all of the gates in the gate-level netlist into rows on the
    chip and then to generate the metal wires that connect all of the
    gates together. We need to provide Cadence Innovus with the same
    abstract logical and timing views used in Synopsys DC, but we also
    need to provide Cadence Innovus with technology information in
    `.lef`, and `.captable` format and abstract physical views of the
    standard-cell library also in `.lef` format. Cadence Innovus will
    generate an updated Verilog gate-level netlist, a `.spef` file which
    contains parasitic resistance/capacitance information about all nets
    in the design, and a `.gds` file which contains the final layout. The
    `.gds` file can be inspected using the open-source Klayout GDS
    viewer. Cadence Innovus also generates reports which can be used to
    accurately characterize area and timing.

 5. We use **Synopsys PrimeTime (PT)** to perform power analysis of our
    design. We need to provide Synopsys PT with the same abstract
    logical, timing, and power views used in Synopsys DC and Cadence
    Innovus, but in addition we need to provide switching activity
    information for every net in the design (which comes from the `.saif`
    file), and capacitance information for every net in the design (which
    comes from the `.spef` file). Synopsys PT puts the switching
    activity, capacitance, clock frequency, and voltage together to
    estimate the power consumption of every net and thus every module in
    the design, and these estimates are captured in various reports.

Extensive documentation is provided by Synopsys and Cadence for these
ASIC tools. We have organized this documentation and made it available to
you on the public course webpage:

 - <https://www.csl.cornell.edu/courses/ece6745/asicdocs>

The first step is to source the setup script, clone this repository from
GitHub, and define an environment variable to keep track of the top
directory for the project.

```bash
% source setup-ece6745.sh
% mkdir -p $HOME/ece6745
% cd $HOME/ece6745
% git clone git@github.com:cornell-ece6745/ece6745-tut06-asic-front-end tut06
% cd tut06
% export TOPDIR=$PWD
% cd $TOPDIR/asic/build-sort
% export ASICDIR=$PWD
```

1. PyMTL3-Based Testing, Simulation, Translation
--------------------------------------------------------------------------

Our goal in this tutorial is to generate layout for the sort unit from
the Verilog tutorial using the ASIC tools. As a reminder, the sort unit
takes as input four integers and a valid bit and outputs those same four
integers in increasing order with the valid bit. The sort unit is
implemented using a three-stage pipelined, bitonic sorting network and
the datapath is shown below.

![](img/tut06-sort-unit-dpath.png)

Let's start by running the tests for the sort unit and note that the
tests for the `SortUnitStruct` will fail.

```bash
% mkdir -p $TOPDIR/sim/build
% cd $TOPDIR/sim/build
% pytest ../tut3_verilog/sort
```

You can just copy over your implementation of the `MinMaxUnit` from when
you completed the Verilog tutorial. If you have not completed the Verilog
tutorial then you might want to go back and do that now. Basically the
`MinMaxUnit` should look like this:

```python
module tut3_verilog_sort_MinMaxUnit
#(
  parameter p_nbits = 1
)(
  input  logic [p_nbits-1:0] in0,
  input  logic [p_nbits-1:0] in1,
  output logic [p_nbits-1:0] out_min,
  output logic [p_nbits-1:0] out_max
);

  always_comb begin

    // Find min/max

    if ( in0 >= in1 ) begin
      out_max = in0;
      out_min = in1;
    end
    else if ( in0 < in1 ) begin
      out_max = in1;
      out_min = in0;
    end

    // Handle case where there is an X in the input

    else begin
      out_min = 'x;
      out_max = 'x;
    end

  end

endmodule
```

Once you have your design working rerun the tests with the
`--test-verilog` and `--dump-vtb` command line options.

```bash
% pytest ../tut3_verilog/sort --test-verilog --dump-vtb
```

The `--test-verilog` and `--dump-vtb` command line options tells the
PyMTL3 framework to dump a Verilog testbench. While PyMTL3 enables
combining Python testbenches with Verilator Verilog simulation, we need
to translate our testbenches to Verilog so that we can use Synopsys VCS
to do 4-state and gate-level simulation. Let's look at a testbench cases
file generated from using the `--dump-vtb` flag.

```bash
% cd $TOPDIR/sim/build
% cat SortUnitStruct__p_nbits_8_test_basic_tb.v.cases

 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
 `T('h04,'h02,'h03,'h01,'h1,'h00,'h00,'h00,'h00,'h0);
 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
 `T('h00,'h00,'h00,'h00,'h0,'h01,'h02,'h03,'h04,'h1);
 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
 `T('h00,'h00,'h00,'h00,'h0,'h00,'h00,'h00,'h00,'h0);
```

This file is generated by logging the inputs and outputs of the Verilator
RTL simulation each cycle. It will be passed into a Verilog testbench
runner that will use these values to set the inputs each cycle and to
verify the outputs each cycle. So note that when we utilize these
testbenches later on, we are running a simulation that is simply
confirming that we acheive the same behavior as the Verilator RTL
simulation we ran using PyMTL3, and it is not actually using any
assertions you wrote in your Python tests for your design. Therefore, it
is important that your RTL simulations pass using PyMTL3 and Verilator
before you move on to other simulations. Also take a look at the
testbench itself to get a sense for how it works. It essentially
instantiates your top module as `DUT`, sets the inputs, and performs a
check every cycle on the outputs.

```bash
% less SortUnitStruct__p_nbits_8_test_basic_tb.v
```

After running the tests we use the sort unit simulator to do the final
evaluation.

```bash
% cd $TOPDIR/sim/build
% ../tut3_verilog/sort/sort-sim --impl rtl-struct --stats --translate --dump-vtb
num_cycles          = 106
num_cycles_per_sort = 1.06
```

Take a moment to open up the translated Verilog which should be in a file
named `SortUnitStruct__p_nbits_8__pickled.v`. The Verilog module name
includes a suffix to make it unique for a specific set of parameters.

2. Using Synopsys VCS for 4-state RTL simulation
-------------------------------------------------------------------------

Using the PyMTL simulation framework can give us a good foundation in
verifying a design. However, the Verilator RTL simulator is only a
2-state simulation, meaning a signal can only be `0` or `1`. An
alternative form of RTL simulation is a 4-state simulation, in which
signals can be `0`, `1`, `x`, or `z`.

It is important to note a key difference between 2-state and 4-state
simulation. In 2-state simulation, each variable is initialized to a
predetermined value. This initial condition assumption may or may not be
what happens in actual silicon! As a result, a different initial
condition could introduce a bug that was not caught by our 2-state
Verilator RTL simulation. In 4-state simulations no such assumptions are
made. Instead, every signal begins as `x`, and only resolves to a `0` or
`1` after it is driven or resolved using x-propagation. Consider the
following pseudocode:

```verilog
always @(*)
begin
  if ( control_signal )
    // set signal "signal_a", but bug causes chip to fail
  else
    // set signal "signal_a" such that everything works fine
end
```

If `control_signal` is not reset, then in 2-state simulation if you
initialize all state to zero it will look like the chip works fine, but
this is not a safe assumption! The real chip does not guarantee that all
state is initialized to zero, so we can model that in four state
simulation as an `x`. Since the control signal could initialize to 1,
this could non-deterministically cause the chip to fail! What you would
see in simulation is that `signal_a` would become an `x`, because we do
not know the value of `control_signal` on reset. This `x` is propagated
through the design, and some simulators are more optimistic/pessimistic
about x's than others. For example, a pessimistic simulator may just
assume that any piece of logic that has an x on the input, outputs an x.
This is pessimistic because it is *possible* that you can still resolve
the output (imagine a mux where two inputs are the same but the select
bit is an `x`). Optimism is the opposite, resolving signals to `0` or `1`
that should remain an `x`.

If your design is passing every 2-state simulation, but failing every
4-state simulation, it may be because invalid fields are being set to
`x`'s. Our test harnesses require all outputs to always be `0` or `1`
even if a field is invalid. So you may need to force invalid fields to
zero and ensure that during a correct execution the outputs of your
module are never `x`'s. You can see this in the implementation of
`SortUnitStruct`:

```verilog
assign out_val = val_S3;
assign out0    = elm0_S3         & {p_nbits{val_S3}};
assign out1    = mmuA_out_min_S3 & {p_nbits{val_S3}};
assign out2    = mmuA_out_max_S3 & {p_nbits{val_S3}};
assign out3    = elm3_S3         & {p_nbits{val_S3}};
```

To create a 4-state simulation, let's start by creating another build
directory for our Synopsys VCS work.

```bash
% cd $ASICDIR/01-synopsys-vcs-rtlsim
```

We run Synopsys VCS to compile a simulation, and `./simv` to run the
simulation. Let's run a 4-state simulation for `test_basic` using the
design `SortUnitStruct__p_nbits_8__pickled.v`.

```bash
% cd $ASICDIR/01-synopsys-vcs-rtlsim
% vcs -sverilog +lint=all -xprop=tmerge -override_timescale=1ns/1ps \
   +incdir+$TOPDIR/sim/build \
   +vcs+dumpvars+SortUnitStruct__p_nbits_8_test_basic_vcs.vcd \
   -top SortUnitStruct__p_nbits_8_tb \
   $TOPDIR/sim/build/SortUnitStruct__p_nbits_8_test_basic_tb.v \
   $TOPDIR/sim/build/SortUnitStruct__p_nbits_8__pickled.v
% ./simv
```

Here some of the key command line options for Synopsys VCS:

```
-sverilog                           indicates we are using SystemVerilog
+lint=all                           turn on all linting checks
-xprop=tmerge                       use more advanced X propoagation
-override_timescale=1ns/1ps         changes the timescale. Units/precision
+incdir+$TOPDIR/sim/build           specifies directories to search for `include
+vcs+dumpvars+filename.vcd          dump VCD in current dir with the name filename.vcd
-top SortUnitStruct__p_nbits_8_tb   name of the top module (located within the VTB)
```

Synopsys VCS is a sophisticated tool with many command line options. If
you want to learn more on your own about other options that are available
to you with Synopsys VCS, you can look at the user guides on the course
webpage:

 - <https://www.csl.cornell.edu/courses/ece6745/asicdocs>

Let's run another 4-state simulation, this time using the testbench from
the sort-rtl simulator run that we ran earlier. Note that while we can
use this VCD for power analysis, for the purposes of this tutorial we
will only be doing power analysis using the gate-level netlist.

```bash
% cd $ASICDIR/01-synopsys-vcs-rtlsim
% vcs -sverilog +lint=all -xprop=tmerge -override_timescale=1ns/1ps \
   +incdir+$TOPDIR/sim/build \
   +vcs+dumpvars+SortUnitStruct__p_nbits_8_sort-rtl-struct-random_vcs.vcd \
   -top SortUnitStruct__p_nbits_8_tb \
   $TOPDIR/sim/build/SortUnitStruct__p_nbits_8_sort-rtl-struct-random_tb.v \
   $TOPDIR/sim/build/SortUnitStruct__p_nbits_8__pickled.v
% ./simv
```

To simplify rerunning a simulation, we can put the above command lines in
a shell script. We have done this for you so you can run it as follows:

```bash
% cd $ASICDIR
% source 01-synopsys-vcs-rtlsim/run.sh
```

3. Using Synopsys Design Compiler for Synthesis
--------------------------------------------------------------------------

We use Synopsys Design Compiler (DC) to synthesize Verilog RTL models
into a gate-level netlist where all of the gates are from the standard
cell library. So Synopsys DC will synthesize the Verilog `+` operator
into a specific arithmetic block at the gate-level. Based on various
constraints it may synthesize a ripple-carry adder, a carry-look-ahead
adder, or even more advanced parallel-prefix adders.

We start by creating a subdirectory for our work, and then launching
Synopsys DC.

```bash
% cd $ASICDIR/02-synopsys-dc-synth
% dc_shell-xg-t
```

To make it easier to copy-and-paste commands from this document, we tell
Synopsys DC to ignore the prefix `dc_shell>` using the following:

```
dc_shell> alias "dc_shell>" ""
```

There are two important variables we need to set before starting to work
in Synopsys DC. The `target_library` variable specifies the standard
cells that Synopsys DC should use when synthesizing the RTL. The
`link_library` variable should search the standard cells, but can also
search other cells (e.g., SRAMs) when trying to resolve references in our
design. These other cells are not meant to be available for Synopsys DC
to use during synthesis, but should be used when resolving references.
Including `*` in the `link_library` variable indicates that Synopsys DC
should also search all cells inside the design itself when resolving
references.

```
dc_shell> set_app_var target_library "$env(ECE6745_STDCELLS)/stdcells.db"
dc_shell> set_app_var link_library   "* $env(ECE6745_STDCELLS)/stdcells.db"
```

Note that we can use `$env(ECE6745_STDCELLS)` to get access to the
`$ECE6745_STDCELLS` environment variable which specifies the directory
containing the standard cells, and that we are referencing the abstract
logical and timing views in the `.db` format.

As an aside, if you want to learn more about any command in any Synopsys
tool, you can simply type `man toolname` at the shell prompt. We are now
ready to read in the Verilog file which contains the top-level design and
all referenced modules. We do this with two commands. The `analyze`
command reads the Verilog RTL into an intermediate internal
representation. The `elaborate` command recursively resolves all of the
module references starting from the top-level module, and also infers
various registers and/or advanced data-path components.

```
dc_shell> analyze -format sverilog $env(TOPDIR)/sim/build/SortUnitStruct__p_nbits_8__pickled.v
dc_shell> elaborate SortUnitStruct__p_nbits_8
```

We need to create a clock constraint to tell Synopsys DC what our target
cycle time is. Synopsys DC will not synthesize a design to run "as fast
as possible". Instead, the designer gives Synopsys DC a target cycle time
and the tool will try to meet this constraint while minimizing area and
power. The `create_clock` command takes the name of the clock signal in
the Verilog (which in this course will always be `clk`), the label to
give this clock (i.e., `ideal_clock1`), and the target clock period in
nanoseconds. So in this example, we are asking Synopsys DC to see if it
can synthesize the design to run at 3.33GHz (i.e., a cycle time of
300ps).

```
dc_shell> create_clock clk -name ideal_clock1 -period 0.3
```

In an ideal world, all inputs and outputs would change immediately with
the clock edge. In reality, this is not the case. We need to include
reasonable delays for inputs and outputs, so Synopsys DC can factor this
into its timing analysis so we would still meet timing if we were to tape
our design out in real silicon. Here, we choose 5% of the clock period
for our input and output delays.

```
dc_shell> set_input_delay  -clock ideal_clock1 [expr 0.3*0.05] [all_inputs]
dc_shell> set_output_delay -clock ideal_clock1 [expr 0.3*0.05] [all_outputs]
```

Next, we give Synopsys DC some constraints about fanout and transition
slew. Fanout roughly describes the number of inputs driven by a
particular output, and the higher the fanout, the higher the drive
strength required. Slew rate is how quickly a signal can make a full
transition. We want all of our signals to meet a good slew, meaning that
they can transition quickly, so we set maximum slew to one quarter of the
clock period.

```
dc_shell> set_max_fanout 20 SortUnitStruct__p_nbits_8
dc_shell> set_max_transition [expr 0.25*0.3] SortUnitStruct__p_nbits_8
```

We can use the `check_design` command to make sure there are no obvious
errors in our Verilog RTL.

```
dc_shell> check_design
```

It is _critical_ that you carefully review all warnings and errors when
you analyze and elaborate a design with Synopsys DC. There may be many
warnings, but you should still skim through them. Often times there will
be something very wrong in your Verilog RTL which means any results from
using the ASIC tools is completely bogus. Synopsys DC will output a
warning, but Synopsys DC will usually just keep going, potentially
producing a completely incorrect gate-level model!

Finally, the `compile` command will do the synthesis.

```
dc_shell> compile
```

During synthesis, Synopsys DC will display information about its
optimization process. It will report on its attempts to map the RTL into
standard-cells, optimize the resulting gate-level netlist to improve the
delay, and then optimize the final design to save area.

The `compile` command does not _flatten_ your design. Flatten means to
remove module hierarchy boundaries; so instead of having module A and
module B within module C, Synopsys DC will take all of the logic in
module A and module B and put it directly in module C. You can enable
flattening with the `-ungroup_all` option. Without extra hierarchy
boundaries, Synopsys DC is able to perform more optimizations and
potentially achieve better area, energy, and timing. However, an
unflattened design is much easier to analyze, since if there is a module
A in your RTL design that same module will always be in the synthesized
gate-level netlist.

The `compile` command does not perform many optimizations. Synopsys DC
also includes `compile_ultra` which does many more optimizations and will
likely produce higher quality of results. Keep in mind that the `compile`
command _will not_ flatten your design by default, while the
`compile_ultra` command _will_ flattened your design by default. You can
turn off flattening by using the `-no_autoungroup` option with the
`compile_ultra` command. `compile_ultra` also has the option
`-gate_clock` which automatically performs clock gating on your design,
which can save quite a bit of power. Once you finish this tutorial, feel
free to go back and experiment with the `compile_ultra` command.

Now that we have synthesized the design, we output the resulting
gate-level netlist in two different file formats: Verilog and `.ddc`
(which we will use with Synopsys DesignVision). We also output an `.sdc`
file which contains the constraint information we gave Synopsys DC. We
will pass this same constraint information to Cadence Innovus during the
place and route portion of the flow.

```
dc_shell> write -format verilog -hierarchy -output post-synth.v
dc_shell> write -format ddc     -hierarchy -output post-synth.ddc
dc_shell> write_sdc -nosplit post-synth.sdc
```

We can use various commands to generate reports about area, energy, and
timing. The `report_timing` command will show the critical path through
the design. Part of the report is displayed below. Note that this report
was generated using a clock constraint of 300ps.

```
dc_shell> report_timing -nosplit -transition_time -nets -attributes
 ...
 Point                                       Fanout Trans Incr  Path
 --------------------------------------------------------------------------
 clock ideal_clock1 (rise edge)                           0.00  0.00
 clock network delay (ideal)                              0.00  0.00
 v/elm2_S2S3/q_reg[2]/CK (DFF_X1)                   0.00  0.00  0.00 r
 v/elm2_S2S3/q_reg[2]/Q (DFF_X1)                    0.01  0.09  0.09 r
 v/elm2_S2S3/q[2] (net)                      3            0.00  0.09 r
 v/elm2_S2S3/q[2] (vc_Reg_p_nbits8_2)                     0.00  0.09 r
 v/elm2_S3[2] (net)                                       0.00  0.09 r
 v/mmuA_S3/in1[2] (MinMaxUnit_p_nbits8_1)           0.00        0.09 r
 v/mmuA_S3/in1[2] (net)                                   0.00  0.09 r
 v/mmuA_S3/U25/ZN (INV_X1)                          0.01  0.02  0.12 f
 v/mmuA_S3/n11 (net)                         1            0.00  0.12 f
 v/mmuA_S3/U3/ZN (NAND2_X1)                         0.01  0.02  0.14 r
 v/mmuA_S3/n8 (net)                          1            0.00  0.14 r
 v/mmuA_S3/U20/ZN (NAND4_X1)                        0.02  0.04  0.18 f
 v/mmuA_S3/net7323 (net)                     1            0.00  0.18 f
 v/mmuA_S3/U11/ZN (OAI221_X1)                       0.05  0.06  0.24 r
 v/mmuA_S3/net7315 (net)                     2            0.00  0.24 r
 v/mmuA_S3/U13/ZN (AND2_X2)                         0.03  0.07  0.31 r
 v/mmuA_S3/net7572 (net)                     8            0.00  0.31 r
 v/mmuA_S3/U53/Z (MUX2_X1)                          0.01  0.08  0.40 f
 v/mmuA_S3/out_min[0] (net)                  1            0.00  0.40 f
 v/mmuA_S3/out_min[0] (MinMaxUnit_p_nbits8_1)       0.00        0.40 f
 v/mmuA_out_min_S3[0] (net)                               0.00  0.40 f
 v/U30/ZN (AND2_X1)                                 0.01  0.03  0.43 f
 v/out1[0] (net)                             1            0.00  0.43 f
 v/out1[0] (SortUnitStruct_p_nbits8)                0.00        0.43 f
 out1[0] (net)                                            0.00  0.43 f
 out1[0] (out)                                      0.01  0.00  0.43 f
 data arrival time                                              0.43

 clock ideal_clock1 (rise edge)                           0.30  0.30
 clock network delay (ideal)                              0.00  0.30
 output external delay                                   -0.01  0.29
 data required time                                             0.29
 ---------------------------------------------------------------------------------------------------------
 data required time                                             0.29
 data arrival time                                             -0.43
 ---------------------------------------------------------------------------------------------------------
 slack (VIOLATED)                                              -0.15
```

This timing report uses _static timing analysis_ to find the critical
path. Static timing analysis checks the timing across all paths in the
design (regardless of whether these paths can actually be used in
practice) and finds the longest path. For more information about static
timing analysis, consult Chapter 1 of the [Synopsys Timing Constraints
and Optimization User
Guide](http://www.csl.cornell.edu/courses/ece6745/asicdocs/tcoug.pdf).
The report clearly shows that the critical path starts at bit 2 of a
pipeline register in between the S2 and S3 stages (`elm2_S2S3`), goes
into an input of a `MinMaxUnit`, comes out the `out_min` port of the
`MinMaxUnit`, and ends at a top-level output port (`out1`). The report
shows the delay through each logic gate (e.g., the clk-to-q delay of the
initial DFF is 90ps, the propagation delay of a NAND2_X1 gate is 20ps)
and the total delay for the critical path which in this case is 0.43ns.
We set the clock constraint to be 300ps, but also notice that the report
factors in the output delay we set with the `set_output_delay` command.

The difference between the required arrival time and the actual arrival
time is called the _slack_. Positive slack means the path arrived before
it needed to while negative slack means the path arrived after it needed
to. If you end up with negative slack, then you need to rerun the tools
with a longer target clock period until you can meet timing with no
negative slack. The process of tuning a design to ensure it meets timing
is called "timing closure". In this course, we are primarily interested
in design-space exploration as opposed to meeting some externally defined
target timing specification. So you will need to sweep a range of target
clock periods. **Your goal is to choose the shortest possible clock
period which still meets timing without any negative slack!** This will
result in a well-optimized design and help identify the "fundamental"
performance of the design. Alternatively, if you are comparing multiple
designs, sometimes the best situation is to tune the baseline so it meets
timing and then ensure the alternative designs have similar cycle times.
This will enable a fair comparison since all designs will be running at
the same cycle time.

The `report_area` command can show how much area each module uses and can
enable detailed area breakdown analysis.

```
dc_shell> report_area -nosplit -hierarchy
...
Combinational area:         388.626001
Buf/Inv area:                88.843999
Noncombinational area:      449.273984
Macro/Black Box area:         0.000000
Net Interconnect area:       undefined  (Wire load has zero net area)

Total cell area:            837.899985
Total area:                  undefined

Hierarchical area distribution
------------------------------

                Global      Local
                Cell Area   Cell Area
                ----------  ----------------
Hierarchical    Abs               Non   Black
Cell            Total  %    Comb  Comb  Boxes
--------------- ----- ---- ----- ----- ----  -------------------------------
SortUnitStruct  837.9  100   0.0   0.0  0.0  SortUnitStruct__p_nbits_8
v               837.9  100  37.2   0.0  0.0  tut3_verilog_sort_SortUnitStruct_p_nbits8
v/elm0_S0S1      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_0
v/elm0_S1S2      36.7  4.4   0.0  36.7  0.0  vc_Reg_p_nbits8_8
v/elm0_S2S3      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_4
v/elm1_S0S1      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_11
v/elm1_S1S2      36.7  4.4   0.0  36.7  0.0  vc_Reg_p_nbits8_7
v/elm1_S2S3      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_3
v/elm2_S0S1      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_10
v/elm2_S1S2      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_6
v/elm2_S2S3      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_2
v/elm3_S0S1      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_9
v/elm3_S1S2      36.7  4.4   0.0  36.7  0.0  vc_Reg_p_nbits8_5
v/elm3_S2S3      36.1  4.3   0.0  36.1  0.0  vc_Reg_p_nbits8_1
v/mmuA_S1        71.2  8.5  71.2   0.0  0.0  tut3_verilog_sort_MinMaxUnit_p_nbits8_0
v/mmuA_S2        71.0  8.5  71.0   0.0  0.0  tut3_verilog_sort_MinMaxUnit_p_nbits8_3
v/mmuA_S3        64.9  7.7  64.9   0.0  0.0  tut3_verilog_sort_MinMaxUnit_p_nbits8_1
v/mmuB_S1        72.8  8.7  72.8   0.0  0.0  tut3_verilog_sort_MinMaxUnit_p_nbits8_4
v/mmuB_S2        67.2  8.0  67.2   0.0  0.0  tut3_verilog_sort_MinMaxUnit_p_nbits8_2
v/val_S0S1        5.8  0.7   1.3   4.5  0.0  vc_ResetReg_p_nbits1_0
v/val_S1S2        5.8  0.7   1.3   4.5  0.0  vc_ResetReg_p_nbits1_2
v/val_S2S3        5.8  0.7   1.3   4.5  0.0  vc_ResetReg_p_nbits1_1
--------------- ----- ---- ----- ----- ----  -----------------------------------------
Total                      388.6 449.2  0.0
```

The units are in square micron. The cell area can sometimes be different
from the total area. The total cell area includes just the standard
cells, while the total area can include interconnect area as well. If
available, we will want to use the total area in our analysis. Otherwise
we can just use the cell area. So we can see that the sort unit consumes
approximately 837um^2 of area. We can also see that each pipeline
register consumes about 4-5% of the area, while the `MinMaxUnit`s consume
about ~40% of the area. This is one reason we try not to flatten our
designs, since the module hierarchy helps us understand the area
breakdowns. If we completely flattened the design there would only be one
line in the above table.

The `report_power` command can show how much power each module consumes.
Note that this power analysis is actually not that useful yet, since at
this stage of the flow the power analysis is based purely on statistical
activity factor estimation. Basically, Synopsys DC assumes every net
toggles 10% of the time. This is a pretty poor estimate, so we should
never use this kind of statistical power estimation in this course.

```
dc_shell> report_power -nosplit -hierarchy
```

Finally, we go ahead and exit Synopsys DC.

```
dc_shell> exit
```

Take a few minutes to examine the resulting Verilog gate-level netlist.
Notice that the module hierarchy is preserved and also notice that the
`MinMaxUnit` synthesizes into a large number of basic logic gates.

```bash
% cd $ASICDIR/02-synopsys-dc-synth
% more post-synth.v
```

We can use the Synopsys Design Vision (DV) tool for browsing the
resulting gate-level netlist, plotting critical path histograms, and
generally analyzing our design. Start Synopsys DV and setup the
`target_library` and `link_library` variables as before.

```
% cd $ASICDIR/02-synopsys-dc-synth
% design_vision-xg
design_vision> set_app_var target_library "$env(ECE6745_STDCELLS)/stdcells.db"
design_vision> set_app_var link_library   "* $env(ECE6745_STDCELLS)/stdcells.db"
```

You can use the following steps to open the `.ddc` file generated during
synthesis.

 - Choose _File > Read_ from the menu
 - Open the `post-synth.dcc` file

You can then use the following steps to browse the gate-level schematic.
First select a module in the Logical Hierarchy panel. Then choose
_Schematic > New Schematic View_. You can double click on modules to
expand them. You might also want to try this approach to see the entire
design at once:

 - Select the `SortUnitStruct__p_nbits_8` module in the Logical Hierarchy panel
 - Choose _Select > Cells > Leaf Cells of Selected Cells_ from the menu
 - Choose _Schematic > New Schematic View_ from the menu
 - Choose _Select > Clear_ from the menu

You can use the following steps to view a histogram of path slack, and
also to open a gave-level schematic of just the critical path.

 - Choose _Timing > Path Slack_ from the menu
 - Click _OK_ in the pop-up window
 - Select the left-most bar in the histogram to see list of most critical paths
 - Select one of the paths in the path list to highlight the path in the schematic view

Or you can right click on a path and choose _Path Schematic_ to see just
the gates that lie on the critical path. Notice that there eight levels
of logic (including the register at the start) on the critical path. The
number of levels of logic on the critical path can provide some very
rough first-order intuition on whether or not we might want to explore a
more aggressive clock constraint and/or adding more pipeline stages. If
there are just a few levels of logic on the critical path then our design
is probably very simple (as in this case!), while if there are more than
50 levels of logic then there is potentially room for signficant
improvement. The following screen capture illutrates using Design Vision
to explore the post-synthesis results. While this can be interesting, in
this course, we almost always prefer exploring the post-place-and-route
results, so we will not really use Synopsys DV that often.

![](img/tut06-synopsys-dv-1.png)

You can automate the above steps by putting a sequence of commands in a
`.tcl` file and run Synopsys DC using those commands in one step like
this:

```bash
% cd $ASICDIR/02-synopsys-dc-synth
% dc_shell-xg-t -f run.tcl
```

To further simplify rerunning this step, we can put the above command
line in a shell script. We have done this for you so you can run it as
follows:

```bash
% cd $ASICDIR
% source 02-synopsys-dc-synth/run.sh
```

Try running Synopsys DC with a target clock period of 0.3ns. Then
gradually increase the clock period until your design meets timing. **To
follow along with the tutorial, push the design through synth again using
0.6 ns as your clock constraint, as this is what we will be using for the
rest of the flow.**

4. Using Synopsys VCS for Fast-Functional Gate-Level Simulation
--------------------------------------------------------------------------

Before synthesis, we used Synopsys VCS to do a 4-state simulation. This
time, we'll be using VCS to perform a gate-level simulation, since we now
have a gate-level netlist available to us. Gate-level simulation provides
an advantage over RTL simulation because it more precisely represents the
specification of the true hardware generated by the tools. This sort of
simulation could propogate X's into the design that were not found by the
4-state RTL simulation, and it also verifies that the tools did not
optimize anything away during synthesis. We will use Synopsys VCS to run
our gate-level simulation on the `sort-rtl-struct-random` simulator
testbench:

```bash
% cd $TOPDIR/asic/build-sort/03-synopsys-vcs-ffglsim
% vcs -sverilog +lint=all -xprop=tmerge -override_timescale=1ns/1ps \
   +incdir+$TOPDIR/sim/build \
   +vcs+dumpvars+SortUnitStruct__p_nbits_8_sort-rtl-struct-random_vcs.vcd \
   -top SortUnitStruct__p_nbits_8_tb \
   +delay_mode_zero \
   +define+CYCLE_TIME=0.6 \
   +define+VTB_INPUT_DELAY=0.03 \
   +define+VTB_OUTPUT_ASSERT_DELAY=0.57 \
   $TOPDIR/sim/build/SortUnitStruct__p_nbits_8_sort-rtl-struct-random_tb.v \
   $ECE6745_STDCELLS/stdcells.v \
   ../02-synopsys-dc-synth/post-synth.v
% ./simv
```

Notice there are some differences in the Synopsys VCS command we ran
here, and the one we ran for 4-state RTL simulation. In this version, we
use the gate-level netlist `post-synth.v` instead of the pickled file. We
also include the option `+delay_mode_zero` which tells Synopsys VCS to
run a fast-functional simulation in which no delays are considered. This
is similar to RTL simulation, and you should notice that all signals will
change on the clock edge. We also include the macros `CYCLE_TIME`,
`VTB_INPUT_DELAY` , `VTB_OUTPUT_ASSERT_DELAY`. These values control how
long after the rising edge we change the inputs and how long after the
rising edge we check the outputs.

The `.vcd` file contains information about the state of every net in the
design on every cycle. This can make these `.vcd` files very large and
thus slow to analyze. For average power analysis, we only need to know
the activity factor on each net. We can use the `vcd2saif` tool to
convert `.vcd` files into `.saif` files. An `.saif` file only contains a
single average activity factor for every net.

```bash
% cd $ASICDIR/03-synopsys-vcs-ffglsim
% vcd2saif -input  ./SortUnitStruct__p_nbits_8_sort-rtl-struct-random_vcs.vcd \
           -output ./SortUnitStruct__p_nbits_8_sort-rtl-struct-random.saif
```

To simplify rerunning a simulation and then running `vcd2said`, we can
put the above command lines in a shell script. We have done this for you
so you can run it as follows:

```bash
% cd $ASICDIR
% source 03-synopsys-vcs-ffglsim/run.sh
```

5. To-Do On Your Own
--------------------------------------------------------------------------

Now we can use what you have learned so far to push the GCD unit through
the flow. First, run a simulation of the GCD unit.

```
% cd $TOPDIR/sim/build
% ../tut3_verilog/gcd/gcd-sim --impl rtl --input random --stats --translate --dump-vtb
% ls GcdUnit_noparam__pickled.v
% ls GcdUnit_noparam_gcd-rtl-random_tb.v
% ls GcdUnit_noparam_gcd-rtl-random_tb.v.cases
```

Now create a new ASIC build directory and copy the scripts we used to
push the sort unit through the ASIC front-end flow.

```
% mkdir -p $TOPDIR/asic/build-gcd
% cd $TOPDIR/asic/build-gcd
% export ASICDIR=$PWD

% mkdir -p $ASICDIR/01-synopsys-vcs-rtlsim
% cp $TOPDIR/asic/build-sort/01-synopsys-vcs-rtlsim/run.sh $ASICDIR/01-synopsys-vcs-rtlsim/run.sh

% mkdir -p $ASICDIR/02-synopsys-dc-synth
% cp $TOPDIR/asic/build-sort/02-synopsys-dc-synth/run.tcl $ASICDIR/02-synopsys-dc-synth
% cp $TOPDIR/asic/build-sort/02-synopsys-dc-synth/run.sh  $ASICDIR/02-synopsys-dc-synth

% mkdir -p $ASICDIR/03-synopsys-vcs-ffglsim
% cp $TOPDIR/asic/build-sort/03-synopsys-vcs-ffglsim/run.sh $ASICDIR/03-synopsys-vcs-ffglsim
```

Now open up each of these files and modify so they push the GCD unit
instead of the sort unit through the flow. For example, the top module
name for the test harness is `GcdUnit_noparam_tb`. You will need to
change the Verilog files used for simulation appropriately. You will need
to change the top-level module you are elaborating during synthesis to
`GcdUnit_noparam` and the cycle-time constraint to 1.0ns. Once you have
updated the scripts you can then push the GCD unit through the flow like
this:


```bash
% cd $ASICDIR
% source 01-synopsys-vcs-rtlsim/run.sh
% source 02-synopsys-dc-synth/run.sh
% source 03-synopsys-vcs-ffglsim/run.sh
```

Carefully look at the post-synthesis gate-level netlist in `post-synt.v`
and the results from running the fast-functional gate-level simulation to
verify that the GCD unit was successfully pushed through the ASIC
front-end flow.

