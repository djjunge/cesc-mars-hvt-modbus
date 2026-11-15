CESC Mars HVT inverter: Modbus registers
Registers of a CESC Mars HV three-phase inverter (model text "Mars 20000 HVT"), read over Modbus TCP. The manufacturer publishes no register map for it. The registers were found by comparing values with the CESC app, the inverter screen and an external P1 meter, and then matched with the N3 HV Modbus protocol v1.00, a closely related map. It fits this unit in the 10000, 11000 to 11097, 14000 and 20100 ranges. The Solinteg hybrid map uses a different address layout and does not fit (see the end).
Verified on one unit, firmware unknown. Status words used below:
Confirmed: matched against the app, the screen or the P1 meter.
Matches N3: the name comes from the N3 HV map and the value on this unit is plausible, not independently checked.
Observed: seen to behave as described, meaning unproven.
Reading the inverter
Item	Value
Transport	Modbus TCP, port 502, unit id 1 (through the Wi-Fi dongle)
Function	3 only (read holding registers)
Clients	One at a time, the dongle refuses a second one
Block size	At most 60 registers per request, pause a little between requests
Read only. Never use function 6 or 16. Writing can change grid settings and void the warranty.
Values are 16 bit unless noted. "Signed" means two's complement (65535 is -1). "x0.1" means divide the raw value by 10.
Live values
Register	Content	Scale and unit	Status
11000	PV1 voltage	x0.1 V	Confirmed
11001	PV1 current	x0.1 A	Confirmed
11002	PV1 power	W	Confirmed
11003 to 11005	PV2 voltage, current, power	same as PV1	Confirmed
11016	Temperature, internal	x0.1 °C	Confirmed against the app
11017	Temperature, charger	x0.1 °C	Confirmed
11018	Temperature, boost	x0.1 °C	Confirmed
11019	Temperature, inverter	x0.1 °C	Confirmed
11076 to 11078	Grid voltage, phases R, S, T	x0.1 V	Confirmed
11079 to 11081	Inverter current, phases R, S, T	x0.1 A, signed	Matches N3
11082 to 11084	Inverter AC power, phases R, S, T	W, signed	Confirmed. R, S, T = L1, L2, L3 is assumed
11085 to 11087	Grid frequency, phases R, S, T	x0.01 Hz	Confirmed
11088 to 11090	EPS (backup output) voltage, reads the same as the grid voltage while on grid	x0.1 V	Matches N3
11091 to 11093	EPS current, reads 0 or 1 (0.1 A, offset noise, nothing connected)	x0.1 A, signed	Matches N3
11094 to 11096	EPS power, reads about -1 to -5 while running, 0 when off	W, signed	Matches N3
11097	EPS frequency, reads about 5000	x0.01 Hz	Matches N3
Battery registers
The unit this file was made on has no battery, so the battery values read offset noise and nothing here could be tested with a real battery. The names, scales and bits below come from the N3 HV map. Treat them as "probably right, unverified". Signed means two's complement.
Register	Content	Scale and unit	Seen on this unit
11020	Battery voltage	x0.1 V	1.7 to 8.2 V, offset noise
11021	Battery current	x0.1 A, signed	0 to 0.2 A, offset noise
11022	Battery power	W, signed	0
11023	BMS voltage	x0.1 V, signed	0
11024	BMS current	x0.1 A, signed	0
11025	BMS temperature	x0.1 C, signed	0
11026, 11027	BMS state of charge, state of health	%	0
11028, 11029	Charge cutoff voltage, discharge cutoff voltage	x0.1 V	0
11030, 11031	Maximum charge current, maximum discharge current	x0.1 A	0
11032, 11033	Highest cell voltage, lowest cell voltage	x0.001 V	0
11034, 11035	Cell number of the highest and lowest voltage	number	0
11036, 11037	Highest cell temperature, lowest cell temperature	x0.1 C	0
11038, 11039	Cell number of the highest and lowest temperature	number	0
11040	BMS basic status. Bits 2 to 0: 0 sleep, 1 charge, 2 discharge, 3 idle. Bit 3 forced charge request, bit 4 balance request	bits	0
11041	BMS cycle count	number	0
11042, 11043	Highest and lowest module voltage	x0.1 V	0
11044, 11045	Module number of the highest and lowest voltage	number	0
11046, 11047	Highest and lowest module temperature	x0.1 C	0
11048, 11049	Module number of the highest and lowest temperature	number	0
11050, 11051	Forbid charge flag, forbid discharge flag	0 or 1	0
11052, 11053, 11054	Total cells, modules, cells per module	number	0
11055	System voltage level	V	0
11056	Battery capacity	Ah	0
11058	BMS connect state		0 with no battery
State
Register	Content	Status
11057	Inverter state: 0 wait, 1 check, 2 normal, 3 standby, 4 fault, 5 permanent, 6 update, 7 EPS check, 8 EPS, 9 self-test, 10 idle	Matches N3. Observed 0, 1 and 2 in one restart, the others come from the map. State 1 lasts the configured grid-check time, 300 s on this unit
11058	BMS connect state	Matches N3. Reads 0 with no battery
11059	Meter 1 connect state	Matches N3. Reads 1 with Meter 1 installed
11060 to 11075	Fault bitfields 1 to 16	Matches N3. Read 0, bits not decoded
11008 to 11010	PV3 (MPPT3) voltage x0.1 V, current x0.1 A, power W	Confirmed against the inverter screen (13.1 V, 0.1 A, 0.00 kW with raw 132, 1, 2). Nothing connected, so only offset noise. Reserved in the N3 map
On this model 11057 is the status register. Register 10105, which holds the status in the Solinteg hybrid map, stayed 0 here, also while generating.
Start-up sequence, logged every 12 seconds while switching the inverter off and on with the SYS switch:
Step	11057	AC power	Remark
Running	2	359 W	Normal
Switched off	0	0	Wait, about 18 seconds, PV3 noise 11008 to 11010 at 0
Self-check	1	0	About 294 seconds measured, 300 seconds as set on the inverter, strings at open-circuit voltage
Connected	2	458 W	Back to normal, 11021 reads 2 for about 10 seconds
The inverter keeps answering Modbus in every state. The value overnight has not been logged yet, 0 (wait) is expected.
Meter 1 (CT clamps on the grid connection)
Needs the meter installed and enabled on the inverter. Without it these registers stay at 0.
Register	Content	Scale and unit	Status
11127	Meter 1 power, phase R	W, signed, import positive	Confirmed (app 88 W, register 85 W)
11129	Meter 1 power, phase S	W, signed	Confirmed (jumped to 1887 W when a load on L2 switched on)
11131	Meter 1 power, phase T	W, signed	Confirmed
11133	Meter 1 total power	W, signed	Confirmed, equals R + S + T exactly (1887 - 95 - 154 = 1638)
11128, 11130, 11132	Between the powers, read 0 or -1		Unknown. Not the high words of 32 bit values, the numbers do not fit
20101	Setting, Meter 1 enabled, 1 = enabled		Matches N3
The N3 HV map puts Meter 1 power at 11098 to 11101, Meter 2 at 11102 to 11105, Meter 3 at 11106 to 11109 and load power at 11110 to 11113. On this inverter all of those read 0, and Meter 1 sits at 11127 to 11133.
Energy counters
Totals are 32 bit, two registers, high word first, x0.1 kWh. "Today" values are one register, x0.1 kWh, and reset at midnight. All read on this unit.
Registers	Content	Status
14000 and 14001, 14002	PV yield total, today	Confirmed
14003 and 14004, 14005	Charge energy total, today	N3 map. 0 with no battery
14006 and 14007, 14008	Discharge energy total, today	N3 map. 0 with no battery
14009 and 14010, 14011	Feed-in total, today	N3 map. 0 while the export limit is 0 W
14012 and 14013, 14014	Consumption total, today	N3 map. Counts, plausible
14015 and 14016, 14017	Output energy total, today	N3 map. Counts, plausible. Lower than the PV yield
14018 and 14019, 14020	Input energy total, today	N3 map. 0 here
14021 and 14022, 14023	Load energy total, today	N3 map. Counts, plausible
14024 and 14025, 14026	EPS (backup output) energy total, today	N3 map. 0 here
Identity
Register	Content	Status
10000 to 10006	Model text, "Mars 20000 HVT", 2 characters per register	Confirmed
10016 to 10023	Serial number, text	Confirmed
20000	System on/off, 0 off, 1 on	Confirmed against the app
20001 to 20006	Inverter clock: year (add 2000), month, day, hour, minute, second	Confirmed, matches the real time
20008	Grid code, position in the app list counted from 0, see the list below. 24 = IEC61727-IN-W	Confirmed against the app
20009 to 20011, 20012	Modbus address of Meter 1 to 3 (1, 2, 3) and of the inverter (1)	Confirmed against the app
11114	Inverter master firmware, 52 = v0.52	Confirmed against the inverter screen
11115	Inverter slave firmware, 100 = v1.00	Confirmed against the inverter screen
11116	Inverter display firmware, 108 = v1.08	Confirmed against the inverter screen
11117 to 11119	Read 0. The screen shows the BMS versions as N/A, so 11117 and 11118 are probably the BMS master and slave versions	Not proven
The N3 map puts the firmware versions at 10176 to 10182. Read as a block this range is refused, but read register by register it answers: 10176 to 10178 hold the same versions as 11114 to 11116 (52, 100, 108), 10181 reads 19 (protocol, N3 name) and 10182 reads 100 (display boot, N3 name). Both places can be used.
Settings (read only)
All values below were checked one by one against the CESC app pages (Safety Voltage, Safety Frequency, P(f), P(u), Reactive Power, UVRT/OVRT, ROCOF, Basic Settings, Function). The grid code on this unit is IEC61727-IN-W. Never write these registers over Modbus, they change grid protection. The N3 map says gain 0.1 for the protection times, on this unit the time in ms is the raw value x 10.
Registers	Content	Scale and unit	Seen on this unit
20100 to 20109	Switches: DRM0, Meter 1, remote power control, ISO, GFCI, anti-islanding, freq protect restrictive, ground check, phase unbalance, bypass relay	0 off, 1 on	Meter 1, ISO, GFCI, anti-islanding on, the rest off
22100	Voltage protection enabled (bits: level 1, level 2, 10 min average)	bits	7
22101, 22103	Overvoltage level 1 and 2	x0.1 V	265 V, 275 V
22105, 22107	Undervoltage level 1 and 2	x0.1 V	180 V, 70 V
22102, 22104, 22106, 22108	Trip times of the four levels	raw x 10 ms	1500, 150, 10000, 1500 ms
22109	10 minute average voltage limit	x0.1 V	258 V
22110	Voltage protection active (bits: over 1, over 2, under 1, under 2, 10 min)	bits	31
22200	Frequency protection enabled (bits: level 1, level 2)	bits	1
22201, 22203	Overfrequency level 1 and 2	x0.01 Hz	52 Hz, 52 Hz
22205, 22207	Underfrequency level 1 and 2	x0.01 Hz	47 Hz, 47 Hz
22202, 22204, 22206, 22208	Trip times of the four levels	raw x 10 ms	150, 150, 1500, 1500 ms
22209	Frequency protection active (bits: over 1, over 2, under 1, under 2)	bits	15
23000 to 23010	P(f): enable, mode, enter delay (ms), over start, over droop (%), over recover, under start, under droop (%), under recover, recover gradient (x0.1 %/min), recover delay (s)	x0.01 Hz where a frequency	off, 50.2 Hz start, 49.7 Hz under start
23100 to 23108	P(u): enable, V1 to V4 (x0.1 V), P1, P4 (%), delay (s), decline gradient		off, 216, 224, 253, 258 V
23200	Reactive power mode: 0 none, 1 fixed PF, 2 fixed Q, 3 cos phi(P), 4 Q(U)		0
23201, 23202	Fixed PF (x0.01), fixed Q (var)		1.00, 0
23203, 23204	Cos phi(P) lock in and lock out voltage	x0.1 V	241.5 V, 225.4 V
23205 to 23212	Cos phi(P) points PF1, P1 to PF4, P4	PF x0.01, P in %	1.00/50, 0.90/100, 0/0, 0/0
23213 to 23215	Q(U) enter delay (s), lock in power, lock out power (%)		1, 20, 5
23216 to 23223	Q(U) points V1, Q1 to V4, Q4	V in % of Un, Q x0.01 %	93/4.4, 97/0, 103/0, 107/4.4
23300, 23301	UVRT and OVRT enable	0 off, 1 on	off, off
23302 to 23307	UVRT V1, T1, V2, T2, V3, T3	V in % of Un, T raw x 10 ms	70/2700, 30/1300, 0/300
23308 to 23313	OVRT V1, T1, V2, T2, V3, T3	same	125/1500, 120/1500, 115/1500
23314, 23315	Zero current under and over voltage threshold	% of Un	85, 110
23400, 23401	ROCOF enable, threshold	x0.1 Hz/s	off, 0.5 Hz/s
Reading tip, important: do not read a block that runs past the last register the inverter implements. On this unit such a block answers without an error but with leftover bytes in its first words, so the values look real and are wrong (for example 22201 read 43.09 Hz instead of 52.00 Hz). Read exactly the ranges in the tables below, and read anything beyond them one register at a time. Registers the inverter does not implement then answer with an error, which is safe. Read exactly, everything from 20000 to 23401 answers, including 10176 to 10182, 20000 to 20013 and 21000 to 21021. Refused (error answer): 10186 to 10199, 11158 to 11179, 20014 to 20019, 21023 to 21045, 22009, 22111 to 22116, 22210 to 22215, 23011 to 23016, 23110 to 23115, 23226 to 23229, 23317 to 23321 and 23402 to 23407.
Settings the app shows that are not in the N3 map, found by reading past the N3 block ends one register at a time and matching the app values:
Register	Setting	Value read	App shows
20110	Hot standby	1	On
20111	Meter 2 enable	0	Off
20112	Meter 3 enable	0	Off
20113	Battery model	254	254
20123	Main breaker current limit (A)	100	100 A
23109	P(u) Tau (s)	5	5 s
23224	Q(U) Tau (s)	5	5 s
23225	Q(U) minimum cos phi, x0.01	40	0.4
23316	Zero current mode	0	Off
Still not found: RCR, meter model (Donghong SDM series), CT direction (facing machine) and the generator settings. The registers 20114 to 20139 hold values the app does not explain: 20114 reads 254, 20121 reads 50, 20126 to 20129 read 20, 80, 480 and 30, the rest 0. Only 20110 to 20113, 20123 and the Tau and cos phi values were matched with the app, the zero current mode only by its value 0.
Work mode, battery and generator settings
None of these could be read over Modbus on the test unit (the dongle refuses 21000 and up), so the N3 names are unverified. The values are from the CESC app pages Working Mode, Function and Gen Settings, which show a battery system that is not installed. Never write these registers.
Registers	Content	N3 range	Seen in the app on this unit
21000	Work mode: 0 self use, 1 force time use, 2 back up, 3 feed-in first		Self use
21001 to 21005	Charge period 1: grid charge flag, start hour, start minute, end hour, end minute		
21006 to 21010	Charge period 2, same layout		
21011 to 21015	Backup charge: grid charge flag, start and end time		
21016, 21017	Maximum set charge and discharge current (x0.1 A)	0 to 30 A	50 A, so the app range is wider than the N3 map
21018, 21019	Minimum SoC, minimum SoC on grid	5 to 100 %	5 %, 5 %
21020	Export limit (W)	0 to 60000	0 W
21021	Power limit, signed percent of rated power	-100 to 100	100
20110 to 20139	See the table above. 20110 to 20113 and 20123 are matched, the rest is unknown	not in the N3 map	see above
Generator settings (Gen Settings page). They are not in the N3 map and the addresses are not known:
App field	Range	Seen
Generator enable	on or off	off
Generator switch on SoC	0 to 100 %	20
Generator switch off SoC	0 to 100 %	80
Generator max run time	30 to 2000 min	480
Generator down time	10 to 1000 min	30
Generator charge enable	on or off	off
Generator charge max SoC	1 to 100 %	15
Generator charge limit power	0 to 65535 W	5000
Generator charge time 1	start and end time	00:00 to 00:00
Commands: the N3 map has write-only command registers at 24000 to 24004, and the app has a Command page. They are not documented here on purpose. Writing them can switch the inverter off or reset it.
Open question: PV power against AC power
Summed PV power (11002 + 11005) is higher than summed AC power (11082 to 11084) by roughly 300 W, almost constant through the day. A restart test next to a P1 meter suggests the AC figure is closer. The inverter's own yield counter (14002) follows the PV figure, but may not be independent of it. Unresolved, check against your own meter.
Grid code list (20008)
The register holds the position in the app dropdown, counted from 0. This unit reads 24, which is IEC61727-IN-W, the same as the app. The list was read from the app, 58 entries. Treat other numbers as read from the list order, only 24 is proven on this unit.
Value	Grid code
0	VDE4105-DE
1	CEI0-21
2	AS4777_A
3	RD1699
4	EN50549
5	EN50549-DK-W
6	Greece
7	EN50549-NL
8	C10/11
9	G99
10	China
11	VDE0126-FR
12	EN50549-PL
13	Brazil-180s
14	VDE0126-DE
15	CEI0-16
16	G98
17	Greece Island
18	EN50549-CZ
19	IEC61727-IN
20	Korea
21	EN50549-SW
22	China-W
23	China-H
24	IEC61727-IN-W
25	Brazil
26	IEC61727-SL
27	Mexico
28	NZ4777
29	Philippines
30	IEC61727-SL-W
31	PEA
32	PEA-W
33	IEC61627-VN
34	IEC61627-VN-W
35	Tunisia
36	MEA
37	MEA-W
38	Brazil-LV
39	EN50549-DK-E
40	Tunisia-W
41	Chile
42	Brazil-W
43	EN50549-PL-W
44	Brazil-180s-W
45	UNE217002-ES
46	G98-NI
47	G99-NI
48	EN50549-NW
49	EN50549-NW-LV
50	IEC61727-LV
51	EN50549-IR
52	Austria
53	AS4777_B
54	AS4777_C
55	EN50549-SIST
56	EN50549-HUN
57	Pakistan
Fault log
Read 30100 first for the number of entries, then 30101 to 30140. Each entry is 4 registers.
Register in entry	High byte	Low byte
1	Year (add 2000)	Month
2	Day	Hour
3	Minute	Second
4	Fault id	Detail code
The time stamp is the inverter's own clock.
Example: raw 6667, 3590, 5410, 3342 is 0x1A0B, 0x0E06, 0x1522, 0x0D0E, so 2026-11-14 06:21:34, fault 13, detail 14.
Verified on two episodes: grid faults with the AC off, then HW Protect and PV OCP during live DC work.
Not verified: the log holds at most 10 entries, and it is unknown whether the oldest drops off or the order differs once it is full.
The N3 map does not list the 30100 range. It is specific to this inverter.
Fault ids
Names as in the CESC Mars HVT user manual V5.0, section 10.1. Causes and remedies are in the manual.
Inverter, grid and PV
Id	Name
001	HW Protect
002	Grid Failure
003	Grid Volt Fault
004	Grid Freq Fault
005	PV-V Fault
006	Bus Volt Fault
008	Grid 10-min OV
009	DCI OCP
012	RC OCP
013	Iso Check Fault
014	Over TEMP Fault
016	Sample Fault
018	Over Load Fault
019	PV ConnDir Fault
023	Ground Fault
025	M/S COM Fault
026	EXT Fan Fault
027	HCT Dev Fault
028	Inv EEPROM Fault
029	RCD Fault
031	Grid Relay Fault
032	Other Dev Fault
035	Inv OCP Fault
036	PV OCP Fault
038	Inv Relay Fault
057	PLL Overtime
061	Parallel Fault
097	HMI EEPROM Fault
099	H/M COM Fault
Smart meter (CT)
Id	Name
098	Meter1 COM Fault
100	Meter2 COM Fault
101	Meter3 COM Fault
Battery and backup output (battery models only)
Id	Name
007	Bat Volt Fault
015	BMS DC Reverse
017	Eps Overload
020	Bat Low Fault
024	BMS COM Fault
033	Eps OCP Fault
037	Bat OCP Fault
114	BMS TEMP Fault
115	BMS InCom Fault
117	BMS PreChg Fault
118	BMS Relay Fault
120	BMS CMU Address
121	BMS Volt Fault
122	BMS OutCOM Fault
123	BMS ISO Fault
124	BMS Check Fault
125	BMS AFE Fault
126	CMU SW Ver. diff
129	BMS UV
130	BMS OV
133	BMS CHG TEMP Low
134	BMS CHG TEMP High
135	BMS DSG TEMP Low
136	BMS DSG TEMP High
137	BMS Charge OC
138	BMS Discharge OC
139	BMS Prot TemDiff
140	BMS Prot AD-AFE
141	BMS Prot Hard OC
142	SOH degradation
143	REPT PROT
Not found yet
The meaning of 11128, 11130, 11132, 11006 and 11007 (2048 and 0), the addresses of the settings the N3 map does not have, and the load power register.
Finding a new register
Change one thing in the real world and see which register follows it: take a snapshot of a range, change the load clearly, take a second snapshot and list the registers that moved. A register is the one you want only if it moves in two separate tests and matches the app. Registers that jitter by 1 or 2 are noise. For 32 bit counters look for two neighbouring registers that read as one number, high word first.
Sources
N3 HV Modbus protocol v1.00 (PDF), the map that fits this inverter
Solinteg Hybrid Inverter Modbus RTU protocol (PDF), a different address layout. Only the status values (0 wait, 1 self-check, 2 on-grid) match, at register 10105 there and 11057 here
CESC Mars HVT user manual V5.0
Not affiliated with CESC, Solinteg or the N3 HV vendor. Use at your own risk.
