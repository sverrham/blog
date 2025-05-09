---
layout: post
title: "VHDL Verilog mixed mode error"
date: 2025-05-09
---

# VHDL/Verilog Questasim error

I had some issues with VCS where it was just totally failing my simulations, so I decided to simulate using Questasim to simulate my design.
The design was a mix of VHDL and Verilog modules and compilation/simulation was controlled by the buildsystem at the company.
Since the buildsystem vas configured and supported many simulators I thought there should be no issues changing simulator.
Sadly this was not the case.
Changing simulator was a quick line change in one file, but the result was error when trying to run the simulation.
The VCOM and VLOG compile steps worked with no issues, but VSIM failed with an error message that did not tell me much.
```
** Fatal: Unexpected signal: 11.
```

Recreating the issue with some example code and testing with Questa Intel Starter FPGA Edition.64 2021.2 I get the following error message when trying to simulate the design.
```vsim -voptargs="+acc" work.testb ```
With the resulting error:
```
# ** Error (suppressible): testb.vhd(24): (vopt-1271) Bad default binding for component instance "u_testa: testa".
#  (Component generic "TESTA" is not on the entity.)
```

This simulates a vhdl module that instantiate a verilog module.
Vhdl module:

```
library ieee;
use ieee.std_logic_1164.all;

entity testb is
end testb;

architecture testb_arch of testb is
  signal clk  : std_logic := '0';
  signal outa : std_logic;

  component testa is
    generic (
      TESTA : integer := 0
    );
    port (
      clk  : in std_logic;
      outa : out std_logic
    );
  end component;

begin

  u_testa : testa
    generic map (
      TESTA => 2
    )
    port map (
      clk  => clk,
      outa => outa 
   );

end architecture;
```

The verilog module is:
```
module testa #(parameter TESTA = 1)

( 
  input wire clk,
  output logic  outa);

always @(clk) begin
  if (clk) begin
    outa = 0;
  end
end

endmodule
```

### Compilation
Verilog module:
```
vlog -sv -lint -93 -work work .\testa.sv
Questa Intel Starter FPGA Edition-64 vlog 2021.2 Compiler 2021.04 Apr 14 2021

vlog -sv -lint -93 -work work .\testa.sv
-- Compiling module testa

Top level modules:
        testa

Errors: 0, Warnings: 0
```

Vhdl module:
```
vcom -2008 -lint -work ./work testb.vhd
Questa Intel Starter FPGA Edition-64 vcom 2021.2 Compiler 2021.04 Apr 14 2021

vcom -2008 -lint -work ./work testb.vhd
-- Loading package STANDARD
-- Loading package TEXTIO
-- Loading package std_logic_1164
-- Compiling entity testb
-- Compiling architecture testb_arch of testb

Errors: 0, Warnings: 0
```

Then simulation:
```
vsim -voptargs=+acc work.testb
# vsim -voptargs="+acc" work.testb 

# ** Note: (vsim-3812) Design is being optimized...
# ** Error (suppressible): testb.vhd(24): (vopt-1271) Bad default binding for component instance "u_testa: testa".
#  (Component generic "TESTA" is not on the entity.)
# Optimization failed
# ** Note: (vsim-12126) Error and warning message counts have been restored: Errors=1, Warnings=0.
# Error loading design

# Errors: 1, Warnings: 0
```

### The issue/solution

The issue was that the vlog command had the switch -93 set, from the help text for -93
```
vlog -93 -help
# Questa Intel Starter FPGA Edition-64 vlog 2021.2 Compiler 2021.04 Apr 14 2021
# vlog options:
# --------------------------------------------------------------------------------
# -93                             Preserve the case of Verilog module (and
#                                 parameter and port) names in the equivalent
#                                 VHDL entity by using VHDL-1993 extended
#                                 identifiers; this may be useful in
#                                 mixed-language designs.
```

With this the simulation with a verilog module having parameters/generics made the simulation fail.

For this simple example it is probably easy to see what is the issue, also the error message here was a bit clearer, not sure why it was different when I was working on the issue, could be different questasim version or other switches that might cause it to fail differently.

This is one of those things you just have to work at and dig into to figure out.
Issue for me working on this was that I was not familiar with the build system and 
it was a big system that took some time to get into.

Also the error message shown here `(Component generic "TESTA" is not on the entity.)` gives a lot more info than the message I got `Fatal: Unexpected signal: 11.`

