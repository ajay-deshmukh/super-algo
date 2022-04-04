## Mobile and Wireless Communication Protocol(s)

###  1. UDP Wireless protocol

```tcl
set val(chan) Channel/WirelessChannel ;# channel type
set val(prop) Propagation/TwoRayGround ;# radio-propagation model
set val(netif) Phy/WirelessPhy ;# network interface type
set val(mac) Mac/802_11 ;# MAC type
set val(ifq) Queue/DropTail/PriQueue ;# interface queue type
set val(ll) LL ;# link layer type
set val(ant) Antenna/OmniAntenna ;# antenna model
set val(ifqlen) 50 ;# max packet in ifq
set val(nn) 4 ;# number of mobilenodes
set val(rp) DSDV ;# routing protocol
set val(x) 624 ;# X dimension of topography
set val(y) 551 ;# Y dimension of topography
set val(stop) 10.0 ;# time of simulation end
#===================================
# Initialization
#===================================
#Create a ns simulator
set ns [new Simulator]
#Setup topography object
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
create-god $val(nn)
#Open the NS trace file
set tracefile [open out.tr w]
$ns trace-all $tracefile
#Open the NAM trace file
set namfile [open out.nam w]
$ns namtrace-all $namfile
$ns namtrace-all-wireless $namfile $val(x) $val(y)
set chan [new $val(chan)];#Create wireless channel
#===================================
# Mobile node parameter setup
#===================================
$ns node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
-channel $chan \
-topoInstance $topo \
-agentTrace ON \
-routerTrace ON \
-macTrace ON \
-movementTrace ON
#===================================
# Nodes Definition
#===================================
#Create 4 nodes
set n0 [$ns node]
$n0 set X_ 318
$n0 set Y_ 375
$n0 set Z_ 0.0
$ns initial_node_pos $n0 20
set n1 [$ns node]
$n1 set X_ 456
$n1 set Y_ 373
$n1 set Z_ 0.0
$ns initial_node_pos $n1 20
set n2 [$ns node]
$n2 set X_ 500
$n2 set Y_ 232
$n2 set Z_ 0.0
$ns initial_node_pos $n2 20
set n3 [$ns node]
$n3 set X_ 524
$n3 set Y_ 451
$n3 set Z_ 0.0
$ns initial_node_pos $n3 20
#===================================
# Agents Definition
#===================================
#Setup a UDP connection
set udp0 [new Agent/UDP]
$ns attach-agent $n0 $udp0
set null4 [new Agent/Null]
$ns attach-agent $n2 $null4
$ns connect $udp0 $null4
$udp0 set packetSize_ 1500
#Setup a UDP connection
set udp1 [new Agent/UDP]
$ns attach-agent $n3 $udp1
set null3 [new Agent/Null]
$ns attach-agent $n2 $null3
$ns connect $udp1 $null3
$udp1 set packetSize_ 1500
#Setup a UDP connection
set udp2 [new Agent/UDP]
$ns attach-agent $n1 $udp2
set null5 [new Agent/Null]
$ns attach-agent $n2 $null5
$ns connect $udp2 $null5
$udp2 set packetSize_ 1500
#===================================
# Applications Definition
#===================================
#Setup a CBR Application over UDP connection
set cbr0 [new Application/Traffic/CBR]
$cbr0 attach-agent $udp0
$cbr0 set packetSize_ 1000
$cbr0 set rate_ 1.0Mb
$cbr0 set random_ null
$ns at 1.0 "$cbr0 start"
$ns at 2.0 "$cbr0 stop"
#Setup a CBR Application over UDP connection
set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp2
$cbr1 set packetSize_ 1000
$cbr1 set rate_ 1.0Mb
$cbr1 set random_ null
$ns at 1.0 "$cbr1 start"
$ns at 2.0 "$cbr1 stop"
#Setup a CBR Application over UDP connection
set cbr2 [new Application/Traffic/CBR]
$cbr2 attach-agent $udp1
$cbr2 set packetSize_ 1000
$cbr2 set rate_ 1.0Mb
$cbr2 set random_ null
$ns at 1.0 "$cbr2 start"
$ns at 2.0 "$cbr2 stop"
#===================================
# Termination
#===================================
#Define a 'finish' procedure
proc finish {} {
global ns tracefile namfile
$ns flush-trace
close $tracefile
close $namfile
exec nam out.nam &
exit 0
}
for {set i 0} {$i < $val(nn) } { incr i } {
$ns at $val(stop) "\$n$i reset"
}
$ns at $val(stop) "$ns nam-end-wireless $val(stop)"
$ns at $val(stop) "finish"
$ns at $val(stop) "puts \"done\" ; $ns halt"
$ns run
```

###  2. TCP Wired protocol

```tcl
# Create a simulator object
set ns [new Simulator]

# Define different colors
# for data flows (for NAM)
$ns color 1 Green
$ns color 2 Red

# Open the NAM trace file
set nf [open out.nam w]
$ns namtrace-all $nf

# Define a 'finish' procedure
proc finish {} {
	global ns nf
	$ns flush-trace
	
	# Close the NAM trace file
	close $nf
	
	# Execute NAM on the trace file
	exec nam out.nam &
	exit 0
}

# Create four nodes
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]

# Create links between the nodes
$ns duplex-link $n0 $n2 2Mb 10ms DropTail
$ns duplex-link $n1 $n2 2Mb 10ms DropTail
$ns duplex-link $n2 $n3 1.7Mb 20ms DropTail

# Set Queue Size of link (n2-n3) to 10
$ns queue-limit $n2 $n3 10

# Give node position (for NAM)
$ns duplex-link-op $n0 $n2 orient right-down
$ns duplex-link-op $n1 $n2 orient right-up
$ns duplex-link-op $n2 $n3 orient right

# Monitor the queue for link (n2-n3). (for NAM)
$ns duplex-link-op $n2 $n3 queuePos 0.5


# Setup a TCP connection
set tcp [new Agent/TCP]
$tcp set class_ 2
$ns attach-agent $n0 $tcp

set sink [new Agent/TCPSink]
$ns attach-agent $n3 $sink
$ns connect $tcp $sink
$tcp set fid_ 1

# Setup a FTP over TCP connection
set ftp [new Application/FTP]
$ftp attach-agent $tcp
$ftp set type_ FTP


# Setup a UDP connection
set udp [new Agent/UDP]
$ns attach-agent $n1 $udp
set null [new Agent/Null]

$ns attach-agent $n3 $null
$ns connect $udp $null
$udp set fid_ 2

# Setup a CBR over UDP connection
set cbr [new Application/Traffic/CBR]
$cbr attach-agent $udp
$cbr set type_ CBR
$cbr set packet_size_ 1000
$cbr set rate_ 1mb
$cbr set random_ false


# Schedule events for the CBR and FTP agents
$ns at 0.1 "$cbr start"
$ns at 1.0 "$ftp start"
$ns at 4.0 "$ftp stop"
$ns at 4.5 "$cbr stop"

# Detach tcp and sink agents
# (not really necessary)
$ns at 4.5 "$ns detach-agent $n0 $tcp ; $ns detach-agent $n3 $sink"

# Call the finish procedure after
# 5 seconds of simulation time
$ns at 5.0 "finish"

# Print CBR packet size and interval
puts "CBR packet size = [$cbr set packet_size_]"
puts "CBR interval = [$cbr set interval_]"

# Run the simulation
$ns run
```

###  3. TCP Wireless protocol

```tcl
set val(chan) Channel/WirelessChannel ;# channel type
set val(prop) Propagation/TwoRayGround ;# radio-propagation model
set val(netif) Phy/WirelessPhy ;# network interface type
set val(mac) Mac/802_11 ;# MAC type
set val(ifq) Queue/DropTail/PriQueue ;# interface queue type
set val(ll) LL ;# link layer type
set val(ant) Antenna/OmniAntenna ;# antenna model
set val(ifqlen) 50 ;# max packet in ifq
set val(nn) 4 ;# number of mobilenodes
set val(rp) DSDV ;# routing protocol
set val(x) 813 ;# X dimension of topography
set val(y) 499 ;# Y dimension of topography
set val(stop) 10.0 ;# time of simulation end
#===================================
# Initialization
#===================================
#Create a ns simulator
set ns [new Simulator]
#Setup topography object
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
create-god $val(nn)
#Open the NS trace file
set tracefile [open out.tr w]
$ns trace-all $tracefile
#Open the NAM trace file
set namfile [open out.nam w]
$ns namtrace-all $namfile
$ns namtrace-all-wireless $namfile $val(x) $val(y)
set chan [new $val(chan)];#Create wireless channel
#===================================
# Mobile node parameter setup
#===================================
$ns node-config -adhocRouting $val(rp) \
 -llType $val(ll) \
 -macType $val(mac) \
 -ifqType $val(ifq) \
 -ifqLen $val(ifqlen) \
 -antType $val(ant) \
 -propType $val(prop) \
 -phyType $val(netif) \
 -channel $chan \
 -topoInstance $topo \
 -agentTrace ON \
 -routerTrace ON \
 -macTrace ON \
 -movementTrace ON
#===================================
# Nodes Definition
#===================================
#Create 4 nodes
set n0 [$ns node]
$n0 set X_ 310
$n0 set Y_ 385
$n0 set Z_ 0.0
$ns initial_node_pos $n0 20
set n1 [$ns node]
$n1 set X_ 713
$n1 set Y_ 399
$n1 set Z_ 0.0
$ns initial_node_pos $n1 20
set n2 [$ns node]
$n2 set X_ 526
$n2 set Y_ 335
$n2 set Z_ 0.0
$ns initial_node_pos $n2 20
set n3 [$ns node]
$n3 set X_ 521
$n3 set Y_ 143
$n3 set Z_ 0.0
$ns initial_node_pos $n3 20
#===================================
# Agents Definition
#===================================
#Setup a TCP connection
set tcp0 [new Agent/TCP]
$ns attach-agent $n0 $tcp0
set sink4 [new Agent/TCPSink]
$ns attach-agent $n2 $sink4
$ns connect $tcp0 $sink4
$tcp0 set packetSize_ 1500
#Setup a TCP connection
set tcp1 [new Agent/TCP]
$ns attach-agent $n3 $tcp1
set sink5 [new Agent/TCPSink]
$ns attach-agent $n2 $sink5
$ns connect $tcp1 $sink5
$tcp1 set packetSize_ 1500
#Setup a TCP connection
set tcp2 [new Agent/TCP]
$ns attach-agent $n1 $tcp2
set sink3 [new Agent/TCPSink]
$ns attach-agent $n2 $sink3
$ns connect $tcp2 $sink3
$tcp2 set packetSize_ 1500
#===================================
# Applications Definition
#===================================
#Setup a FTP Application over TCP connection
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ns at 1.0 "$ftp0 start"
$ns at 2.0 "$ftp0 stop"
#Setup a FTP Application over TCP connection
set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp1
$ns at 1.0 "$ftp1 start"
$ns at 2.0 "$ftp1 stop"
#Setup a FTP Application over TCP connection
set ftp2 [new Application/FTP]
$ftp2 attach-agent $tcp2
$ns at 1.0 "$ftp2 start"
$ns at 2.0 "$ftp2 stop"
#===================================
# Termination
#===================================
#Define a 'finish' procedure
proc finish {} {
 global ns tracefile namfile
 $ns flush-trace
 close $tracefile
 close $namfile
 exec nam out.nam &
 exit 0
}
for {set i 0} {$i < $val(nn) } { incr i } {
 $ns at $val(stop) "\$n$i reset"
}
$ns at $val(stop) "$ns nam-end-wireless $val(stop)"
$ns at $val(stop) "finish"
$ns at $val(stop) "puts \"done\" ; $ns halt"
$ns run
```

###  4. Mobile Ad-hoc Network Protocol

```tcl
set val(chan) Channel/WirelessChannel ;# channel type
set val(prop) Propagation/TwoRayGround ;# radio-propagation model
set val(netif) Phy/WirelessPhy ;# network interface type
set val(mac) Mac/802_11 ;# MAC type
set val(ifq) Queue/DropTail/PriQueue ;# interface queue type
set val(ll) LL ;# link layer type
set val(ant) Antenna/OmniAntenna ;# antenna model
set val(ifqlen) 50 ;# max packet in ifq
set val(nn) 6 ;# number of mobilenodes
set val(rp) DSDV ;# routing protocol
set val(x) 1283 ;# X dimension of topography
set val(y) 100 ;# Y dimension of topography
set val(stop) 10.0 ;# time of simulation end
#===================================
# Initialization
#===================================
#Create a ns simulator
set ns [new Simulator]
#Setup topography object
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
create-god $val(nn)
#Open the NS trace file
set tracefile [open out.tr w]
$ns trace-all $tracefile
#Open the NAM trace file
set namfile [open out.nam w]
$ns namtrace-all $namfile
$ns namtrace-all-wireless $namfile $val(x) $val(y)
set chan [new $val(chan)];#Create wireless channel
#===================================
# Mobile node parameter setup
#===================================
$ns node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
-channel $chan \
-topoInstance $topo \
-agentTrace ON \
-routerTrace ON \
-macTrace ON \
-movementTrace ON
#===================================
# Nodes Definition
#===================================
#Create 6 nodes
set n0 [$ns node]
$n0 set X_ 323
$n0 set Y_ 184
$n0 set Z_ 0.0
$ns initial_node_pos $n0 20
set n1 [$ns node]
$n1 set X_ 126
$n1 set Y_ 234
$n1 set Z_ 0.0
$ns initial_node_pos $n1 20
set n2 [$ns node]
$n2 set X_ 341
$n2 set Y_ 380
$n2 set Z_ 0.0
$ns initial_node_pos $n2 20
set n3 [$ns node]
$n3 set X_ 568
$n3 set Y_ 214
$n3 set Z_ 0.0
$ns initial_node_pos $n3 20
set n4 [$ns node]
$n4 set X_ 427
$n4 set Y_ 5
$n4 set Z_ 0.0
$ns initial_node_pos $n4 20
set n5 [$ns node]
$n5 set X_ 175
$n5 set Y_ -17
$n5 set Z_ 0.0
$ns initial_node_pos $n5 20
#===================================
# Agents Definition
#===================================
#Setup a TCP connection
set tcp5 [new Agent/TCP]
$ns attach-agent $n1 $tcp5
set sink0 [new Agent/TCPSink]
$ns attach-agent $n0 $sink0
$ns connect $tcp5 $sink0
$tcp5 set packetSize_ 1500
#Setup a TCP connection
set tcp6 [new Agent/TCP]
$ns attach-agent $n2 $tcp6
set sink1 [new Agent/TCPSink]
$ns attach-agent $n0 $sink1
$ns connect $tcp6 $sink1
$tcp6 set packetSize_ 1500
#Setup a TCP connection
set tcp7 [new Agent/TCP]
$ns attach-agent $n3 $tcp7
set sink2 [new Agent/TCPSink]
$ns attach-agent $n0 $sink2
$ns connect $tcp7 $sink2
$tcp7 set packetSize_ 1500
#Setup a TCP connection
set tcp8 [new Agent/TCP]
$ns attach-agent $n4 $tcp8
set sink3 [new Agent/TCPSink]
$ns attach-agent $n0 $sink3
$ns connect $tcp8 $sink3
$tcp8 set packetSize_ 1500
#Setup a TCP connection
set tcp9 [new Agent/TCP]
$ns attach-agent $n5 $tcp9
set sink4 [new Agent/TCPSink]
$ns attach-agent $n0 $sink4
$ns connect $tcp9 $sink4
$tcp9 set packetSize_ 1500
#===================================
# Applications Definition
#===================================
#Setup a FTP Application over TCP connection
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp5
$ns at 1.0 "$ftp0 start"
$ns at 2.0 "$ftp0 stop"
#Setup a FTP Application over TCP connection
set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp6
$ns at 1.0 "$ftp1 start"
$ns at 2.0 "$ftp1 stop"
#Setup a FTP Application over TCP connection
set ftp2 [new Application/FTP]
$ftp2 attach-agent $tcp7
$ns at 1.0 "$ftp2 start"
$ns at 2.0 "$ftp2 stop"
#Setup a FTP Application over TCP connection
set ftp3 [new Application/FTP]
$ftp3 attach-agent $tcp8
$ns at 1.0 "$ftp3 start"
$ns at 2.0 "$ftp3 stop"
#Setup a FTP Application over TCP connection
set ftp4 [new Application/FTP]
$ftp4 attach-agent $tcp9
$ns at 1.0 "$ftp4 start"
$ns at 2.0 "$ftp4 stop"
#===================================
# Termination
#===================================
#Define a 'finish' procedure
proc finish {} {
global ns tracefile namfile
$ns flush-trace
close $tracefile
close $namfile
exec nam out.nam &
exit 0
}
for {set i 0} {$i < $val(nn) } { incr i } {
$ns at $val(stop) "\$n$i reset"
}
$ns at $val(stop) "$ns nam-end-wireless $val(stop)"
$ns at $val(stop) "finish"
$ns at $val(stop) "puts \"done\" ; $ns halt"
$ns run
```

###  5. Congestion Control in the computer network

```tcl
set val(stop) 10.0 ;# time of simulation end
#===================================
# Initialization
#===================================
#Create a ns simulator
set ns [new Simulator]
#Open the NS trace file
set tracefile [open out.tr w]
$ns trace-all $tracefile
#Open the NAM trace file
set namfile [open out.nam w]
$ns namtrace-all $namfile
#===================================
# Nodes Definition
#===================================
#Create 12 nodes
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]
set n6 [$ns node]
set n7 [$ns node]
set n8 [$ns node]
set n9 [$ns node]
set n10 [$ns node]
set n11 [$ns node]
#===================================
# Links Definition
#===================================
#Createlinks between nodes
$ns duplex-link $n2 $n0 100.0Mb 10ms DropTail
$ns queue-limit $n2 $n0 50
$ns duplex-link $n1 $n2 100.0Mb 10ms DropTail
$ns queue-limit $n1 $n2 50
$ns duplex-link $n3 $n2 100.0Mb 10ms DropTail
$ns queue-limit $n3 $n2 50
$ns duplex-link $n2 $n4 100.0Mb 10ms DropTail
$ns queue-limit $n2 $n4 50
$ns duplex-link $n5 $n3 100.0Mb 10ms DropTail
$ns queue-limit $n5 $n3 50
$ns duplex-link $n4 $n5 100.0Mb 10ms DropTail
$ns queue-limit $n4 $n5 50
$ns duplex-link $n6 $n7 100.0Mb 10ms DropTail
$ns queue-limit $n6 $n7 50
$ns duplex-link $n6 $n8 100.0Mb 10ms DropTail
$ns queue-limit $n6 $n8 50
$ns duplex-link $n7 $n9 100.0Mb 10ms DropTail
$ns queue-limit $n7 $n9 50
$ns duplex-link $n8 $n9 100.0Mb 10ms DropTail
$ns queue-limit $n8 $n9 50
$ns duplex-link $n9 $n10 0.5Mb 10ms DropTail
$ns queue-limit $n9 $n10 50
$ns duplex-link $n9 $n11 0.5Mb 10ms DropTail
$ns queue-limit $n9 $n11 50
$ns simplex-link $n5 $n6 0.3Mb 10ms DropTail
$ns queue-limit $n5 $n6 50
$ns simplex-link $n6 $n5 0.3Mb 10ms DropTail
$ns queue-limit $n6 $n5 50
#Give node position (for NAM)
$ns duplex-link-op $n2 $n0 orient left-up
$ns duplex-link-op $n1 $n2 orient right-up
$ns duplex-link-op $n3 $n2 orient left-down
$ns duplex-link-op $n2 $n4 orient right-down
$ns duplex-link-op $n5 $n3 orient left-up
$ns duplex-link-op $n4 $n5 orient right-up
$ns duplex-link-op $n6 $n7 orient right-up
$ns duplex-link-op $n6 $n8 orient right-down
$ns duplex-link-op $n7 $n9 orient right-down
$ns duplex-link-op $n8 $n9 orient right-up
$ns duplex-link-op $n9 $n10 orient right-up
$ns duplex-link-op $n9 $n11 orient right-down
$ns simplex-link-op $n5 $n6 orient right
$ns simplex-link-op $n6 $n5 orient left
#===================================
# Agents Definition
#===================================
#Setup a TCP connection
set tcp0 [new Agent/TCP]
$ns attach-agent $n0 $tcp0
set sink4 [new Agent/TCPSink]
$ns attach-agent $n10 $sink4
$ns connect $tcp0 $sink4
$tcp0 set packetSize_ 1500
#Setup a UDP connection
set udp1 [new Agent/UDP]
$ns attach-agent $n1 $udp1
set null3 [new Agent/Null]
$ns attach-agent $n11 $null3
$ns connect $udp1 $null3
$udp1 set packetSize_ 1500
#===================================
# Applications Definition
#===================================
#Setup a FTP Application over TCP connection
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ns at 1.0 "$ftp0 start"
$ns at 2.0 "$ftp0 stop"
#Setup a CBR Application over UDP connection
set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp1
$cbr1 set packetSize_ 1000
$cbr1 set rate_ 1.0Mb
$cbr1 set random_ null
$ns at 1.0 "$cbr1 start"
$ns at 2.0 "$cbr1 stop"
#===================================
# Termination
#===================================
#Define a 'finish' procedure
proc finish {} {
global ns tracefile namfile
$ns flush-trace
close $tracefile
close $namfile
exec nam out.nam &
exit 0
}
$ns at $val(stop) "$ns nam-end-wireless $val(stop)"
$ns at $val(stop) "finish"
$ns at $val(stop) "puts \"done\" ; $ns halt"
$ns run
```

###  6. Clustering between network nodes and data transmission

```tcl
set val(chan) Channel/WirelessChannel ;# channel type

set val(prop) Propagation/TwoRayGround ;#

set val(netif) Phy/WirelessPhy ;# network interface type

set val(mac) Mac/802_11 ;# MAC type

set val(ifq) Queue/DropTail/PriQueue ;# interface queue

set val(ll) LL ;# link layer type

set val(ant) Antenna/OmniAntenna ;# antenna model

set val(ifqlen) 50 ;# max packet in ifq

set val(nn) 6 ;# number of mobilenodes

set val(rp) DSDV ;# routing protocol

set val(x) 1096 ;# X dimension of topography

set val(y) 524 ;# Y dimension of topography

set val(stop) 10.0 ;# time of simulation end

#intiall

#Create a ns simulator

set ns [new Simulator]

#Setup topography object

set topo [new Topography]

$topo load_flatgrid $val(x) $val(y)

create-god $val(nn)


set tracefile [open out.tr w]

$ns trace-all $tracefile

#Open the NAM trace file

set namfile [open out.nam w]

$ns namtrace-all $namfile

$ns namtrace-all-wireless $namfile $val(x) $val(y)

set chan [new $val(chan)];#Create wireless channel


$ns node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
-channel $chan \
-topoInstance $topo \
-agentTrace ON \
-routerTrace ON \
-macTrace ON \
-movementTrace ON

#Create 6 nodes

set n0 [$ns node]

$n0 set X_ 665

$n0 set Y_ 327

$n0 set Z_ 0.0

$ns initial_node_pos $n0 20

set n1 [$ns node]

$n1 set X_ 884

$n1 set Y_ 280

$n1 set Z_ 0.0

$ns initial_node_pos $n1 20

set n2 [$ns node]

$n2 set X_ 474

$n2 set Y_ 424

$n2 set Z_ 0.0

$ns initial_node_pos $n2 20

set n3 [$ns node]

$n3 set X_ 980

$n3 set Y_ 401

$n3 set Z_ 0.0

$ns initial_node_pos $n3 20

set n4 [$ns node]

$n4 set X_ 996

$n4 set Y_ 264

$n4 set Z_ 0.0

$ns initial_node_pos $n4 20



set n5 [$ns node]

$n5 set X_ 509

$n5 set Y_ 290

$n5 set Z_ 0.0

$ns initial_node_pos $n5 20

#Setup a TCP connection

set tcp0 [new Agent/TCP]

$ns attach-agent $n2 $tcp0

set sink8 [new Agent/TCPSink]

$ns attach-agent $n0 $sink8

$ns connect $tcp0 $sink8

$tcp0 set packetSize_ 1500

#==================================

# Agents Definition
#==================================


#Setup a TCP connection

set tcp1 [new Agent/TCP]

$ns attach-agent $n0 $tcp1

set sink13 [new Agent/TCPSink]

$ns attach-agent $n1 $sink13

$ns connect $tcp1 $sink13

$tcp1 set packetSize_ 1500


#Setup a TCP connection

set tcp2 [new Agent/TCP]

$ns attach-agent $n1 $tcp2

set sink10 [new Agent/TCPSink]

$ns attach-agent $n0 $sink10

$ns connect $tcp2 $sink10

$tcp2 set packetSize_ 1500



#Setup a TCP connection

set tcp3 [new Agent/TCP]

$ns attach-agent $n3 $tcp3

set sink11 [new Agent/TCPSink]

$ns attach-agent $n1 $sink11

$ns connect $tcp3 $sink11

$tcp3 set packetSize_ 1500



#Setup a TCP connection

set tcp5 [new Agent/TCP]

$ns attach-agent $n5 $tcp5

set sink9 [new Agent/TCPSink]

$ns attach-agent $n0 $sink9

$ns connect $tcp5 $sink9

$tcp5 set packetSize_ 1500



#Setup a TCP connection

set tcp6 [new Agent/TCP]

$ns attach-agent $n4 $tcp6

set sink12 [new Agent/TCPSink]

$ns attach-agent $n1 $sink12

$ns connect $tcp6 $sink12

#==================================

# Applications Definition
#==================================


#Setup a FTP Application over TCP connection

set ftp0 [new Application/FTP]

$ftp0 attach-agent $tcp0

$ns at 1.0 "$ftp0 start"

$ns at 2.0 "$ftp0 stop"

#==================================

# Applications Definition
#==================================


#Setup a FTP Application over TCP connection

set ftp1 [new Application/FTP]

$ftp1 attach-agent $tcp1

$ns at 1.0 "$ftp1 start"

$ns at 2.0 "$ftp1 stop"

#==================================

# Applications Definition
#==================================


#Setup a FTP Application over TCP connection

set ftp2 [new Application/FTP]

$ftp2 attach-agent $tcp2

$ns at 1.0 "$ftp2 start"

$ns at 2.0 "$ftp2 stop"

#Setup a FTP Application over TCP connection

set ftp3 [new Application/FTP]

$ftp3 attach-agent $tcp3

$ns at 1.0 "$ftp3 start"

$ns at 2.0 "$ftp3 stop"

#==================================

# Applications Definition
#==================================


#Setup a FTP Application over TCP connection

set ftp4 [new Application/FTP]

$ftp4 attach-agent $tcp5

$ns at 1.0 "$ftp4 start"

$ns at 2.0 "$ftp4 stop"

#Setup a FTP Application over TCP connection

set ftp5 [new Application/FTP]

$ftp5 attach-agent $tcp6

$ns at 1.0 "$ftp5 start"

$ns at 2.0 "$ftp5 stop"

#===================================
# Termination
#===================================

#Define a 'finish' procedure

proc finish {} {

global ns tracefile namfile

$ns flush-trace

close $tracefile

close $namfile

exec nam out.nam &

exit 0

}

#==================================

# Termination
#==================================


for {set i 0} {$i < $val(nn) } { incr i } {

$ns at $val(stop) "\$n$i reset"

}

$ns at $val(stop) "$ns nam-end-wireless $val(stop)"

$ns at $val(stop) "finish"

$ns at $val(stop) "puts \"done\" ; $ns halt"

$ns run
```

###  7. Simulation of distance vector routing algorithm

```tcl
set ns [new Simulator]
$ns rtproto LS

set node1 [$ns node]
set node2 [$ns node]

set node3 [$ns node]
set node4 [$ns node]
set node5 [$ns node]

set node6 [$ns node]
set node7 [$ns node]

set tf [open out.tr w]

$ns trace-all $tf

set nf [open out.nam w]

$ns namtrace-all $nf

$node1 label "node 1"

$node1 label "node 2"
$node1 label "node 3"

$node1 label "node 4"
$node1 label "node 5"
$node1 label "node 6"

$node1 label "node 7"
$node1 label-color blue

$node2 label-color red
$node3 label-color red
$node4 label-color blue

$node5 label-color blue
$node6 label-color blue

$node7 label-color blue

$ns duplex-link $node1 $node2 1.5Mb 10ms DropTail
$ns duplex-link $node2 $node3 1.5Mb 10ms DropTail

$ns duplex-link $node3 $node4 1.5Mb 10ms DropTail
$ns duplex-link $node4 $node5 1.5Mb 10ms DropTail
$ns duplex-link $node5 $node6 1.5Mb 10ms DropTail

$ns duplex-link $node6 $node7 1.5Mb 10ms DropTail
$ns duplex-link $node7 $node1 1.5Mb 10ms DropTail

$ns duplex-link-op $node1 $node2 orient left-down
$ns duplex-link-op $node2 $node3 orient left-down

$ns duplex-link-op $node3 $node4 orient right-down
$ns duplex-link-op $node4 $node5 orient right

$ns duplex-link-op $node5 $node6 orient right-up
$ns duplex-link-op $node6 $node7 orient left-up
$ns duplex-link-op $node7 $node1 orient left-up

set tcp2 [new Agent/TCP]
$ns attach-agent $node1 $tcp2
set sink2 [new Agent/TCPSink]

$ns attach-agent $node4 $sink2
$ns connect $tcp2 $sink2

set ftp [new Application/FTP]
$ftp attach-agent $tcp2

proc finish {} {
	global ns nf
	$ns flush-trace
	close $nf
	exec nam out.nam &
	exit 0
	}

$ns at 0.5 "$ftp start"

$ns rtmodel-at 1.0 down $node2 $node3
$ns rtmodel-at 2.0 up $node2 $node3
$ns at 3.0 "$ftp start"

$ns at 4.0 "$ftp stop"
$ns at 5.0 "finish"

$ns run
```

###  8. Simulation of link states Vector routing protocol

```tcl
set ns [new Simulator]

$ns rtproto LS

set nf [open link.nam w]
$ns namtrace-all $nf

proc finish {} {
global ns nf
$ns flush-trace
close $nf
exec nam link.nam &
exit 0
}

for {set i 0} {$i < 5} {incr i} {
set n($i) [$ns node]
}

for {set i 0} {$i < 5} {incr i} {
$ns duplex-link $n($i) $n([expr ($i+1)%5]) 1Mb 10ms DropTail
}

set udp0 [new Agent/UDP]
$ns attach-agent $n(0) $udp0


set cbr0 [new Application/Traffic/CBR]
$cbr0 set packetSize_ 500
$cbr0 set interval_ 0.005
$cbr0 attach-agent $udp0

set null0 [new Agent/Null]
$ns attach-agent $n(2) $null0

$ns connect $udp0  $null0

$ns at 0.5 "$cbr0 start"
$ns rtmodel-at 1.0 down $n(1) $n(2)
$ns rtmodel-at 2.0 up $n(1) $n(2)
$ns at 3.5 "$cbr0 stop"
$ns at 5.0 "finish"

$ns run
```

###  9. Topologies in the nework simulator

#### BUS Topology
```tcl
set ns [new Simulator]
#Open the nam trace file
set nf [open out.nam w]
$ns namtrace-all $nf
#Define a 'finish' procedure
proc finish {} {
 global ns nf
 $ns flush-trace
 #Close the trace file
 close $nf
 #Executenam on the trace file
 exec nam out.nam &
 exit 0
}
#Create five nodes
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
#Create Lan between the nodes
set lan0 [$ns newLan "$n0 $n1 $n2 $n3 $n4" 0.5Mb 40ms LL Queue/DropTail MAC/Csma/Cd Channel]

#Create a TCP agent and attach it to node n0
set tcp0 [new Agent/TCP]
$tcp0 set class_ 1
$ns attach-agent $n1 $tcp0
#Create a TCP Sink agent (a traffic sink) for TCP and attach it to node n3
set sink0 [new Agent/TCPSink]
$ns attach-agent $n3 $sink0
#Connect the traffic sources with the traffic sink
$ns connect $tcp0 $sink0
# Create a CBR traffic source and attach it to tcp0
set cbr0 [new Application/Traffic/CBR]
$cbr0 set packetSize_ 500
$cbr0 set interval_ 0.01
$cbr0 attach-agent $tcp0
#Schedule events for the CBR agents
$ns at 0.5 "$cbr0 start"
$ns at 4.5 "$cbr0 stop"
#Call the finish procedure after 5 seconds of simulation time
$ns at 5.0 "finish"
#Run the simulation
$ns run
```

#### HYBRID Topology
```tcl
set ns [new Simulator]
set nf [open out.nam w]
$ns namtrace-all $nf

proc finish {} {
        global ns nf
        $ns flush-trace
        close $nf
        exec nam out.nam
        exit 0
}

$ns rtproto DV

# ---------------------------------------------------------
# create nodes and establish links
for {set i 1} {$i<5} {incr i} {
set r($i) [$ns node]
set h($i) [$ns node]
}

for {set i 1} {$i<15} {incr i} {
set p($i) [$ns node]
}

#Creating Links between h and p
for {set i 1} {$i<4} {incr i} {
$ns duplex-link $h(1) $p($i) 1.5Mb 10ms SFQ
$ns duplex-link $h(3) $p([expr ($i+7)]) 1.5Mb 10ms SFQ
}

for {set i 4} {$i<8} {incr i} {
$ns duplex-link $h(2) $p($i) 1.5Mb 10ms SFQ
$ns duplex-link $h(4) $p([expr ($i+7)]) 1.5Mb 10ms SFQ
}


#Creating Links between r nodes and connecting r to h
for {set i 1} {$i<4} {incr i} {
$ns duplex-link $r($i) $r([expr ($i+1)]) 1.5Mb 10ms SFQ
$ns duplex-link $r($i) $h($i) 1.5Mb 10ms SFQ
}

$ns duplex-link $r(4) $r(1) 1.5Mb 10ms SFQ
$ns duplex-link $r(4) $h(4) 1.5Mb 10ms SFQ



# orienting the nodes
$ns duplex-link-op $r(1) $h(1) orient up 
$ns duplex-link-op $r(2) $h(2) orient right
$ns duplex-link-op $r(3) $h(3) orient down
$ns duplex-link-op $r(4) $h(4) orient left


# ----------------------------------------------------
#creating tcp agents and attach
set tcp3 [new Agent/TCP] 
set tcp9 [new Agent/TCPSink]

$ns attach-agent $p(3) $tcp3
$ns attach-agent $p(9) $tcp9

$ns connect $tcp3 $tcp9

set tcp5 [new Agent/TCP] 
set tcp12 [new Agent/TCPSink]

$ns attach-agent $p(5) $tcp5
$ns attach-agent $p(12) $tcp12

$ns connect $tcp5 $tcp12

#creating FTP application for tcp agents
set ftp3 [new Application/FTP]
set ftp5 [new Application/FTP]

$ftp3 attach-agent $tcp3
$ftp5 attach-agent $tcp5
# ----------------------------------------------------

#creating udp agents and attach
set udp13 [new Agent/UDP]
set udp6 [new Agent/Null]

$ns attach-agent $p(13) $udp13
$ns attach-agent $p(6) $udp6

$ns connect $udp13 $udp6

set udp1 [new Agent/UDP]
set udp8 [new Agent/Null]

$ns attach-agent $p(1) $udp1
$ns attach-agent $p(8) $udp8

$ns connect $udp1 $udp8


#creating CBR applications for udp
set cbr13 [new Application/Traffic/CBR]

# send 50 packets in 1 second i.e 1 packet every 1/50 second
$cbr13 set packetSize_ 1536
$cbr13 set interval_ 0.02
$cbr13 attach-agent $udp13

set cbr1 [new Application/Traffic/CBR]

# send 400 packets in 1 second i.e 1 packet every 1/400 second
$cbr1 set packetSize_ 5632
$cbr1 set interval_ 0.0025
$cbr1 attach-agent $udp1

# -----------------------------------------------------
# setting events

$ns rtmodel-at 0.7 down $r(1) $r(2)
$ns rtmodel-at 1.0 up $r(1) $r(2)

$ns rtmodel-at 0.9 down $r(4) $r(3)
$ns rtmodel-at 1.3 up $r(4) $r(3)


$ns at 0.2 "$ftp3 start"
$ns at 1.8 "$ftp3 stop"

$ns at 0.3 "$ftp5 start"
$ns at 1.4 "$ftp5 stop"


$ns at 0.4 "$cbr13 start"
$ns at 1.6 "$cbr13 stop"

$ns at 0.7 "$cbr1 start"
$ns at 1.7 "$cbr1 stop"

$ns at 2.0 "finish"
$ns run
```

#### MESH Topology
```tcl
set ns [new Simulator]
#Open the nam trace file
set nf [open out.nam w]
$ns namtrace-all $nf
#Define a 'finish' procedure
proc finish {} {
 global ns nf
 $ns flush-trace
 #Close the trace file
 close $nf
 #Executenam on the trace file
 exec nam out.nam &
 exit 0
}

#Create four nodes
set node1 [$ns node]
set node2 [$ns node]
set node3 [$ns node]
set node4 [$ns node]

#Create links between the nodes
$ns duplex-link $node1 $node2 1Mb 20ms FQ
$ns duplex-link $node1 $node3 1Mb 20ms FQ
$ns duplex-link $node1 $node4 1Mb 20ms FQ
$ns duplex-link $node2 $node3 1Mb 20ms FQ
$ns duplex-link $node2 $node4 1Mb 20ms FQ
$ns duplex-link $node3 $node4 1Mb 20ms FQ

set tcp0 [new Agent/TCP]
$tcp0 set class_ 1
$ns attach-agent $node1 $tcp0
#Creating a TCP Sink agent for TCP and attaching it to  node 3
set sink0 [new Agent/TCPSink]
$ns attach-agent $node3 $sink0
#Connecting the traffic sources with the traffic sink
$ns connect $tcp0 $sink0
# Creating a CBR traffic source and attach it to tcp0
set cbr0 [new Application/Traffic/CBR]
$cbr0 set packetSize_ 500
$cbr0 set interval_ 0.05
$cbr0 attach-agent $tcp0
#Schedule events for the CBR agents
$ns at 0.5 "$cbr0 start"
$ns at 5.5 "$cbr0 stop"
#Here we call the finish procedure after 10 seconds of simulation time
$ns at 10.0 "finish"
#Finally run the simulation
$ns run
```

#### RING Topology
```tcl
set ns [new Simulator]
$ns rtproto DV

set nf [open out.nam w]
$ns namtrace-all $nf

proc finish {} {
        global ns nf
        $ns flush-trace
        close $nf
        exec nam out.nam
        exit 0
        }

#Creating Nodes
for {set i 0} {$i<7} {incr i} {
set n($i) [$ns node]
}

#Creating Links
for {set i 0} {$i<7} {incr i} {
$ns duplex-link $n($i) $n([expr ($i+1)%7]) 512Kb 5ms DropTail
}

$ns duplex-link-op $n(0) $n(1) queuePos 1
$ns duplex-link-op $n(0) $n(6) queuePos 1

#Creating UDP agent and attching to node 0
set udp0 [new Agent/UDP]
$ns attach-agent $n(0) $udp0

#Creating Null agent and attaching to node 3
set null0 [new Agent/Null]
$ns attach-agent $n(3) $null0

$ns connect $udp0 $null0


#Creating a CBR agent and attaching it to udp0
set cbr0 [new Application/Traffic/CBR]
$cbr0 set packetSize_ 1024
$cbr0 set interval_ 0.01
$cbr0 attach-agent $udp0

$ns rtmodel-at 0.4 down $n(2) $n(3)
$ns rtmodel-at 1.0 up $n(2) $n(3)

$ns at 0.01 "$cbr0 start"
$ns at 1.5 "$cbr0 stop"

$ns at 2.0 "finish"
$ns runs
```

#### STAR Topology
```tcl
set ns  [new Simulator]

$ns color 1 green
$ns color 2 yellow

$ns rtproto DV

set nf [open out.nam w]
$ns namtrace-all $nf

proc finish {} {
        global ns nf
        $ns flush-trace
        close $nf
        exec nam out.nam
        exit 0
        }

#creating Nodes        
for {set i 0} {$i<7} {incr i} {
set n($i) [$ns node]
}

#Creating Links
for {set i 1} {$i<7} {incr i} {
$ns duplex-link $n(0) $n($i) 512Kb 10ms SFQ
}

#Orienting The nodes
$ns duplex-link-op $n(0) $n(1) orient left-up
$ns duplex-link-op $n(0) $n(2) orient right-up
$ns duplex-link-op $n(0) $n(3) orient right
$ns duplex-link-op $n(0) $n(4) orient right-down
$ns duplex-link-op $n(0) $n(5) orient left-down
$ns duplex-link-op $n(0) $n(6) orient left

#TCP_Config
set tcp0 [new Agent/TCP]
$tcp0 set class_ 1
$ns attach-agent $n(1) $tcp0

set sink0 [new Agent/TCPSink]
$ns attach-agent $n(4) $sink0

$ns connect $tcp0 $sink0

#UDP_Config
set udp0 [new Agent/UDP]
$udp0 set class_ 2
$ns attach-agent $n(2) $udp0

set null0 [new Agent/Null]
$ns attach-agent $n(5) $null0

$ns connect $udp0 $null0

#CBR Config
set cbr0 [new Application/Traffic/CBR]
$cbr0 set rate_ 256Kb
$cbr0 attach-agent $udp0

#FTP Config
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0


#Scheduling Events
$ns rtmodel-at 0.4 down $n(0) $n(5)
$ns rtmodel-at 0.9 up $n(0) $n(5)

$ns rtmodel-at 0.7 down $n(0) $n(4)
$ns rtmodel-at 1.2 up $n(0) $n(4)

$ns at 0.1 "$ftp0 start"
$ns at 1.5 "$ftp0 stop"

$ns at 0.2 "$cbr0 start"
$ns at 1.3 "$cbr0 stop"

$ns at 2.0 "finish"
$ns run
```
