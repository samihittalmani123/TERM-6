#simulation parameters setup 
set val(chan) Channel/WirelessChannel; 
set val(prop) Propagation/TwoRayGround; 
set val(netif) Phy/WirelessPhy; 
set val(mac) Mac/802_11; 
set val(ifq) CMUPriQueue; 
set val(ll) LL; 
set val(ant) Antenna/OmniAntenna; 
set val(ifqlen) 50; 
set val(nn) 6; 
set val(rp) DSR; 
set val(X) 700; 
set val(Y) 700; 
set val(stop) 60.0; 
#create a ns simulator 
set ns [new Simulator] 
#setup topography object 
set topo [new Topography] 
$topo load_flatgrid $val(X) $val(Y) 
create-god $val(nn) 
#open the NS trace file 
set tracefile [open lab6.tr w] 
$ns trace-all $tracefile 
#open the NS nam file 
set namfile [open lab6.nam w] 
$ns namtrace-all $namfile 
$ns namtrace-all-wireless $namfile $val(X) $val(Y) 
#create wireless channel 
set chan [new $val(chan)]; 
$ns node-config -adhocRouting $val(rp) \ -llType 
$val(ll) \ -macType $val(mac) \ -ifqType 
$val(ifq) \ -ifqLen -antType 
$val(ifqlen) \ 
$val(ant) \ -propType $val(prop) \ -phyType $val(netif) \ -channel 
$chan\ -topoInstance $topo\ -agentTrace ON \ -routerTrace  ON \ -macTrace ON \ -movementTrace ON 
#create node with initial positions 
set n0 [$ns node] 
$n0 set X_ 150 
$n0 set Y_ 300 
$n0 set Z_ 0.0 
$ns initial_node_pos $n0 20 
set n1 [$ns node] 
$n1 set X_ 300 
$n1 set Y_ 500 
$n1 set Z_ 0.0 
$ns initial_node_pos $n1 20 
set n2 [$ns node] 
$n2 set X_ 500 
$n2 set Y_ 500 
$n2 set Z_ 0.0 
$ns initial_node_pos $n2 20 
set n3 [$ns node] 
$n3 set X_ 300 
$n3 set Y_ 100 
$n3 set Z_ 0.0 
$ns initial_node_pos $n3 20 
set n4 [$ns node] 
$n4 set X_ 500 
$n4 set Y_ 100 
$n4 set Z_ 0.0 
$ns initial_node_pos $n4 20 
set n5 [$ns node] 
$n5 set X_ 650 
$n5 set Y_ 300 
$n5 set Z_ 0.0 
$ns initial_node_pos $n5 20 
set tcp0 [new Agent/TCP] 
$ns attach-agent $n0 $tcp0 
set sink5 [new Agent/TCPSink] 
$ns attach-agent $n5 $sink5 
$ns connect $tcp0 $sink5 
$tcp0 set packetSize_ 1500 
set ftp0 [new Application/FTP] 
$ftp0 attach-agent $tcp0 
$ns at 3.0 "$ftp0 start" 
$ns at 60.0 "$ftp0 stop" 
#allow node n3 to move towards node n1 with speed 5m/sec 
$ns at 4.0 "$n3 setdest 300. 500.0 5,0" 
#define a finish function 
proc finish {} { 
global ns tracefile namfile 
$ns flush-trace 
close $tracefile 
close $namfile 
exec nam lab6.nam & 
exec awk -f lab6.awk lab6.tr & 
exec xgraph lab.dat -geometry 800*400 -t "Packets received by each 
wirless node" -x "Node Number" -y "Packet Received" & 
exit 0 
} 
for {set i 0} {$i < $val(nn)} {incr i} { 
$ns at $val(stop) "\$n$i reset" 
} 
$ns at $val(stop) "$ns nam-end-wireless $val(stop)" 
$ns at $val(stop) "finish" 
$ns at $val(stop) "puts\"done\";$ns halt" 
$ns run 


Awk file: 
BEGIN{ 
count1=0; 
count2=0; 
count3=0; 
count4=0; 
count5=0; 
} 
{ 
if( $1=="r" && $3=="_1_" && $4=="RTR") 
count1++; 
if( $1=="r" && $4=="RTR" && $3=="_2_") 
count2++; 
if( $1=="r" && $4=="RTR" && $3=="_3_") 
count3++; 
if( $1=="r" && $4=="RTR" && $3=="_4_") 
count4++; 
if( $1=="r" && $4=="RTR" && $3=="_5_") 
count5++; 
if( $1=="SFESTs") 
printf("\n%lf\t%d\t%s\t%d\t%s\t%s\t%d\t%s\t%s",$2,$5,$6,$7,$11,$12,$1 
3,$14,$15); 
} 
END{ 
} 
printf("\n Packets received by node 1 %d",count1); 
printf("\n Packets received by node 2 %d",count2); 
printf("\n Packets received by node 3 %d",count3); 
printf("\n Packets received by node 4 %d",count4); 
printf("\n Packets received by node 5 %d",count5);
