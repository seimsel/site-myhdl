---
title:  Continuous Sinus Waveform Generator
layout: page
content: []
---

The follwing is a small hardware description which will generate a sinus waveform. This is accomplished by using the recursive chebycheff formulation, thereby only one multiplication per clock-cycle is used.


The plotting of the generated waveform is done using pylab/matplotlib.


Myhdl Code
==========
```python
#!/usr/bin/env python
 
from myhdl import *
from math import cos,pi
import pylab
 
 
def unit_singen(i_clk,i_reset,i_enable,o_outvalue,frequenzSIN,clkfrequnez):
 
    INTERNALWIDTH=len(o_outvalue)-2 
 
    KONSTANT_FACTOR=int(cos(2*pi*frequenzSIN*1.0/clkfrequnez)*2**(INTERNALWIDTH))
 
    Reg_T0=Signal(intbv((2**(INTERNALWIDTH))-1,min=o_outvalue.min,max=o_outvalue.max))
    Reg_T1=Signal(intbv(KONSTANT_FACTOR,min=o_outvalue.min,max=o_outvalue.max))
 
    @always(i_clk.posedge,i_reset.negedge)
    def logicCC():
      if i_reset== 0 :
	Reg_T0.next=(2**(INTERNALWIDTH))-1
	Reg_T1.next=KONSTANT_FACTOR
      else:
	if i_enable==1:
	  # recursive chebycheff formulation for sinus waveform calculation
	  # often taylor approximations or other polynomial approximations 
          # are used, but if you want a continuous waveform,
	  # the recursive chebycheff formulation could save a bunch of multiplications.
	  Reg_T0.next=Reg_T1
	  Reg_T1.next=((KONSTANT_FACTOR * Reg_T1)>>(INTERNALWIDTH-1)) - Reg_T0
 
    @always_comb
    def comb_logic():
      o_outvalue.next=Reg_T1
 
    return instances()
 
 
def test_singen(SimulateNPoints=200):
 
   OUTPUT_BITWIDTH=30
   Sinusvalue=Signal(intbv(0,min=-2**OUTPUT_BITWIDTH,max=2**OUTPUT_BITWIDTH))
   clk=Signal(bool(0))
   enable=Signal(bool(0))
   reset=Signal(bool(0))
   frequenzSIN=0.75e6   # make a 1.45 mhz Sinus
   clk_frequnez=10e6   # 10 mhz 
 
   clk_period=1.0/clk_frequnez
   singen_inst = unit_singen(clk,reset,enable,Sinusvalue,frequenzSIN,clk_frequnez)
 
   @always(delay(int(clk_period*0.5*1e9)))  ## delay in nano seconds
   def clkgen():
       clk.next = not clk
 
 
   sinlist=[]
 
   @instance
   def stimulus():
 
       while 1:
	 reset.next=0
	 enable.next=0
	 yield clk.posedge
	 reset.next=1
	 yield clk.posedge
	 enable.next=1
	 for i in range(SimulateNPoints):
	    yield clk.posedge
	    #Save the results from the simulation
	    sinlist.append(int(Sinusvalue))
 
	 # Plot the Simulation Result using pylab/Mathplotlib
	 pylab.figure(1)
	 pylab.subplot(211)
	 pylab.plot(pylab.arange(0.0,clk_period*(len(sinlist)-0.5),clk_period),sinlist)
	 pylab.xlabel("Time")
	 pylab.ylabel("Amplitude")
	 pylab.subplot(212)
	 pylab.semilogy(pylab.arange(-clk_frequnez/2.0,clk_frequnez/2.0,clk_frequnez/(len(sinlist))),abs(pylab.fftshift(pylab.fft(pylab.array(sinlist)))))
	 pylab.xlabel("Frequency")
	 pylab.ylabel("Amplitude")
	 pylab.show()
 
	 raise StopSimulation
 
   toVHDL(unit_singen,clk,reset,enable,Sinusvalue,frequenzSIN,clk_frequnez)
 
   return instances()
 
 
if __name__ == '__main__':
    sim = Simulation(test_singen(SimulateNPoints=200))
    sim.run()
```

Waveform Plot
=============
![Continuous Sinwave FFT](/users/hardsoftlucid/continuous_sinwave_fft.png)

Conversion to VHDL
==================
```vhdl
-- File: unit_singen.vhd
-- Generated by MyHDL 0.7
-- Date: Tue Mar 27 11:52:47 2012
 
 
library IEEE;
use IEEE.std_logic_1164.all;
use IEEE.numeric_std.all;
use std.textio.all;
 
use work.pck_myhdl_07.all;
 
entity unit_singen is
    port (
        i_clk: in std_logic;
        i_reset: in std_logic;
        i_enable: in std_logic;
        o_outvalue: out signed (30 downto 0)
    );
end entity unit_singen;
 
 
architecture MyHDL of unit_singen is
 
signal Reg_T1: signed (30 downto 0);
signal Reg_T0: signed (30 downto 0);
 
begin
 
 
 
 
 
o_outvalue <= Reg_T1;
 
 
UNIT_SINGEN_LOGICCC: process (i_clk, i_reset) is
begin
    if (i_reset = '0') then
        Reg_T0 <= to_signed((2 ** 29) - 1, 31);
        Reg_T1 <= "0011100100000110010000000011101";
    elsif rising_edge(i_clk) then
        if (i_enable = '1') then
            Reg_T0 <= Reg_T1;
            Reg_T1 <= resize(shift_right((478355485 * Reg_T1), (29 - 1)) - Reg_T0, 31);
        end if;
    end if;
end process UNIT_SINGEN_LOGICCC;
 
end architecture MyHDL;
```
