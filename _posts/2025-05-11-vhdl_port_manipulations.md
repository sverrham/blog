---
layout: post
title: "VHDL portmap signal manipulations" 
date: 2025-05-18
---


I recently needed to change typ of some signals going in to a vhdl module, and had some issues with some functions not working and some did.
So I wanted to go through what I can do and what I can't do when manipulating signals mapped in a portmap of an entity in vhdl.
So I did some tests and at least you can do these things when compiling with vhdl-2008

The usual way is to do an instantiation with the same type so `std_logic_vector` mapped to `std_logic_vector`.

```vhdl
  std_logic_vector => std_logic_vector
  a_data => a_data
 ```

Another thing that I often see is type cast for changing to a different type like `std_logic_vector` from `unsigned` or the other way.

 ```vhdl
 std_logic_vector <= std_logic_vector(unsigned)
 b_data(b_data'range) => unsigned(b_data),
```

It is also possible to make functions and convert or do remapping with it.
```vhdl
function some_function(constant data : std_logic_vector) return std_logic_vector
```
Then this function can be used on the port.

```vhdl
std_logic_vector(range) => some_function(std_logic_vector)
```
The range on the port removes a warning when I compiled with vhdl-2008. I had the port as unconstrained and so setting the range on the output removes a warning in compilation.


It is also possible to change the output which I don't see used that often, like changing `std_logic_vector` to `unsigned`
```vhdl
  unsigned(std_logic_vector(range)) => unsigned
```

One can also run a function on the output for conversion/manipulation
```vhdl
some_function(data_out) => data_out,
```

So these are some possibilities of manipulation of signals directly on the entity instantiation.
One can also access slices by ranging the outputs.


## Attachment
Some example code

Test instantiation
```vhdl

library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

library work;
use work.test_pkg.all;

entity testmodule_inst is
    port (
        clk : in std_logic;
        reset : in std_logic;
        data_in : in std_logic_vector(7 downto 0);
        data_out : out std_logic_vector(7 downto 0)
    );
end entity testmodule_inst;

architecture behavior of testmodule_inst is

  signal a_data : std_logic_vector(7 downto 0);
  signal b_data : std_logic_vector(7 downto 0);
  signal c_data : unsigned(7 downto 0);


  function bit_manip(constant data : std_logic_vector) return std_logic_vector is
    variable result : std_logic_vector(data'range);
  begin
    for i in data'range loop
      result(i) := not data(i);
    end loop;
    return result;
  end function;

  signal d_out : test_record(b(4 downto 0));

begin

  dut : entity work.testmodule
    port map (
        clk => clk,
        reset => reset,
        a_data(a_data'range) => bit_manip(a_data),
        b_data(b_data'range) => unsigned(b_data),
        unsigned(c_data(c_data'range)) => c_data,
        data_in => unsigned(data_in),
        bit_manip(data_out) => data_out,
        d_in => test_record'(a => a_data, b => unsigned(b_data), c => '0'),
        d_out => d_out
    );  


end architecture behavior;
```

Test module
```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

library work;
use work.test_pkg.all;

entity testmodule is
    port (
        clk : in std_logic;
        reset : in std_logic;
        a_data : in std_logic_vector;
        b_data : in unsigned;
        c_data : out std_logic_vector;
        data_in : in unsigned(7 downto 0);
        data_out : out std_logic_vector(7 downto 0);
        d_in : in test_record;
        d_out : out test_record
    );
end entity testmodule;

architecture behavior of testmodule is
    signal internal_signal : std_logic_vector(7 downto 0);

begin
    process(clk, reset)
    begin
        if reset = '1' then
            internal_signal <= (others => '0');
        elsif rising_edge(clk) then
            internal_signal <= std_logic_vector(data_in);
        end if;
    end process;

    data_out <= internal_signal;

    c_data <= std_logic_vector(unsigned(a_data) + b_data);


    process (clk)
    begin
        if rising_edge(clk) then
            d_out <= d_in;
        end if;
    end process;

end architecture behavior;
````

package
```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

package test_pkg is


  type test_record is record 
    a : std_logic_vector(7 downto 0);
    b : unsigned;
    c : std_logic;
  end record;

end package test_pkg;
```