# fantastic-chicken-leg

* SPICE INPUT FILE: Bsim3demo2.sp ID-VGS
.param	Supply=1.8		* Set value of Vdd
.lib	'C:/temp/003/mm018.l' TT	* Set 0.18um library
.opt	scale=0.1u		* Set lambda

*.model pch PMOS level=49 version=3.1
*.model nch NMOS level=49 version=3.1

.global vcc

.subckt nmos n1 n2 n3
mn n1  n2 n3 Gnd 	  nch	l=2 w=6 ad=20 pd=4 as=20 ps=4
.ends nmos

.subckt pmos n1 n2 n3
mp n1 n2 n3 vcc  	  pch	l=2 w=3 ad=20 pd=4 as=20 ps=4
.ends pmos

.subckt inv in out
*vcc vcc gnd 1.8
xmn out in gnd nmos 
xmp out in vcc pmos
.ends inv

.subckt nand2 a b out
*vcc vcc gnd 1.8
xmp1 out a vcc pmos
xmp2 out b vcc pmos
xmn1 out a lab nmos
xmn2 lab b gnd nmos
.ends nand2

.subckt and2 a b out
xnand2 a b out0 nand2
xinv out0 out inv
.ends subckt

.subckt or2 a b y
*vcc vcc gnd 1.8
xmp1 l12 a vcc pmos
xmp2 l24 b l12 pmos
xmp4 l24 a gnd nmos
xmn3 l24 b gnd nmos
xmp5 y l24 vcc pmos
xmp6 y l24 gnd nmos
.ends or2

.subckt nor2 a b out 
xor a b out0 or2
xinv out0 out inv
.ends nor2

.subckt xnor2 a b out 
xnor2a a b oa nor2
xnor2b a oa ob nor2
xnor2c oa b oc nor2
xnor2d ob oc out nor2
.ends xnor2 

*1-b 0-a
.subckt mux2 b a sel out
xnand2a a self oa nand2
xinvb sel self inv
xnandc sel b oc nand2
xnandd oa oc out nand2
.ends mux2

*.subckt latchgroup d s1 s2 mc q

*.ends latchgroup

.subckt clknclr c1 c2 clr s1 s2 mc
xinv1 c1 c1f inv
xinv2 c2 c2f inv
xand2a c1f c2 s1 and2
xand2b c2f c1 t and2
xxnor2a c1 c2 s2 xnor2
xor2a clr t mc or2
.ends clknclr

*xlatgrp d s1 s2 mc q latchgroup
xclknclr c1 c2 clr s1 s2 mc clknclr
xmux2a f1 d s1 m mux2
xand2b m mc f1 and2
xmux2c q f1 s2 s mux2
xand2d s mc q and2

vdclr clr gnd 1.8 

vcc vcc 0 1.8
*Vin in gnd pulse 0 1.8 10n 2n 2n 2000n 4000n
*vclk clk gnd pulse 0 1.8 10n 1n 1n 50n 100n
vc1 c1 gnd pulse 0 1.8 10n 1n 1n 100n 200n
vc2 c2 gnd pulse 0 1.8 10n 1n 1n 200n 400n
vd d gnd pulse 0 1.8 10n 1n 1n 25n 50n

.tran 1n 4000n	

.end
