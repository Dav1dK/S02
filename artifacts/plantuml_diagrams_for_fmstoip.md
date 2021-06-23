# Content #

This files contains FMStoIP PlantUML diagrams that is used the spec, and 

## How to render these plant-uml sequence diagrams ##

Here is the method I use:

1. Install Visual Studio Code
2. Install Visual Studio Code extension "PlantUML"
3. Install Docker
4. Run, in a terminal `docker run -it -p 8080:8080 plantuml/plantuml-server:jetty`
5. In VS Code extensions Change "PlantUML" settings:
   - Renderer: **PlantUMLServer**
   - Server: http://localhost:8080
   - Export Format: **png**
   - Export Out Dir: **./**
6. Symbol in upper right corner for "Open Preview to the side" 
7. The images in preview cannot be copied to Word. Do Ctrl-Shift-P to get the command menu and write/select PlantUML: Export Current File Diagrams
8. Then replace images in word document with image on disk
9. Commit any updated diagrams to git (that are already in git)

# Configuration Service #

```plantuml
@startuml
title Configuration service overview
participant "Client A\nip:1.2.3.4" as ClientA
participant "Client B\nip:3.4.5.6" as ClientB
participant "Client C\nip:1.2.3.4" as ClientC
participant FMStoIP

activate FMStoIP

note over FMStoIP: Data structure used for registrations \nis an implementation detail

ClientA -> FMStoIP: addpgn(ClientId=AcmeCo, F003, EverySecond)
activate ClientA
note over FMStoIP: Registrations\nF003: [("AcmeCo@1.2.3.4", ES)]
FMStoIP -> ClientA: http response: 200 (OK) \nbody: Standard response body SUCCESS
deactivate ClientA

ClientB -> FMStoIP: addpgn(ClientId=EcoDrv, FEAE, EverySecond)
activate ClientB
note over FMStoIP: Registrations\nF003: [("AcmeCo@1.2.3.4", ES)]\nFEAE: [("EcoDrv@3.4.5.6", ES)]
FMStoIP -> ClientB: 200 (OK)
deactivate ClientB

ClientC -> FMStoIP: addpgn(ClientId=BrokenLtd, F003, EverySecond)
activate ClientC
note over FMStoIP: Registrations\nF003: [("AcmeCo@1.2.3.4" ES),("BrokenLtd@1.2.3.4", ES)]\nFEAE: [("EcoDrv@3.4.5.6", ES)]
FMStoIP -> ClientC: http response: 200 (OK) \nbody: Standard response body SUCCESS
deactivate ClientC

ClientB -> FMStoIP: removepgn(ClientId=EcoDrv, F003)
activate ClientB
note over FMStoIP: Registrations\nF003: [("AcmeCo@1.2.3.4" ES),("BrokenLtd@1.2.3.4", ES)]\nFEAE: [("EcoDrv@3.4.5.6", ES)]
FMStoIP -> ClientB: http response: 200 (OK) \nbody: Standard response body SUCCESS
note left of FMStoIP: It is the end result that matter; registration removed \n(since it was not there in the first place.)
deactivate ClientB

ClientB -> FMStoIP: removepgn(ClientId=EcoDrv, FEAE)
activate ClientB
note over FMStoIP: Registrations\nF003: [("AcmeCo@1.2.3.4", ES),("BrokenLtd@1.2.3.4", ES)]\n
FMStoIP -> ClientB: http response: 200 (OK) \nbody: Standard response body SUCCESS
deactivate ClientB

@enduml
```

# Provisioning Service #

```plantuml
@startuml 
title Simple_EverySecond_example
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
clients -> FMSSrv: C1 addpgn(P1,ES)
hnote over clients: dt max 1s
FMSSrv -> clients: FMS<rt=2250, P1(no-value, no-rt)>

BusFMS -> FMSSrv: P1(V1) [rt=2450]
BusFMS -> FMSSrv: P1(V2) [rt=2550]
BusFMS -> FMSSrv: P1(V2) [rt=2650]
...
BusFMS -> FMSSrv: P1(V2) [rt=3150]
hnote over clients: dt 952ms
FMSSrv -> clients: FMS<rt=3202, P1(V2, rt=3150)>
note over clients: 3202 is timestamp of when FMS data sent on multicast\n3150 is timestamp of PGN P1 received by FMStoIP service

BusFMS -> FMSSrv: P1(V2) [rt=3350]
BusFMS -> FMSSrv: P1(V3) [rt=3450]
...
BusFMS -> FMSSrv: P1(V3) [rt=4150]
hnote over clients: dt 973ms
FMSSrv -> clients: FMS<rt=4223, P1(V3, rt=4150)>
...
@enduml
```

### Simple EverySecond and OnChange example ###

```plantuml
@startuml 
title Simple_EverySecond_and_OnChange_example
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
clients -> FMSSrv: C1 addpgn(P1,ES)
hnote over clients: dt max 1s
FMSSrv -> clients: FMS<rt=2250, P1(no-value, no-rt)>


BusFMS -> FMSSrv: P1(V1) [rt=2450]
BusFMS -> FMSSrv: P1(V2) [rt=2550]
clients -> FMSSrv: C2 addpgn(P1,OC)
BusFMS -> FMSSrv: P1(V2) [rt=2650]
note right of FMSSrv: P1(V2) **not** sent, as value does not change.
...
BusFMS -> FMSSrv: P1(V2) [rt=3150]
hnote over clients: dt 952ms
FMSSrv -> clients: FMS<rt=3202, P1(V2, rt=3150)>
note right of FMSSrv: 3202 is timestamp of when FMS data sent on multicast\n3150 is timestamp of PGN P1 received by FMStoIP service

BusFMS -> FMSSrv: P1(V2) [rt=3350]
BusFMS -> FMSSrv: P1(V3) [rt=3450]
hnote over clients: dt 338ms
FMSSrv -> clients: FMS<rt=3540, P1(V3, rt=3450)>
note over clients: Sent 90ms after P1(V3) received
...
BusFMS -> FMSSrv: P1(V3) [rt=4150]
BusFMS -> FMSSrv: P1(V3) [rt=4250]
BusFMS -> FMSSrv: P1(V3) [rt=4350]
BusFMS -> FMSSrv: P1(V3) [rt=4450]
hnote over clients: dt 958ms
FMSSrv -> clients: FMS<rt=4498, P1(V3, rt=4450)>
note right FMSSrv: Send due to value change at rt=3540 have shifted \nES updates compared to ES updates at rt=2250/3202.
...
@enduml
```

### Simple multiple PGNs ###

```plantuml
@startuml 
title Simple_multiple_PGNs
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
clients -> FMSSrv: C1 addpgn(P1,ES)
clients -> FMSSrv: C1 addpgn(P2,ES)
clients -> FMSSrv: C1 addpgn(P3,ES)
FMSSrv -> clients: FMS<rt=2250, P1(no-value, no-rt), P2(no-value, no-rt), P3(no-value, no-rt)>

BusFMS -> FMSSrv: P1(V1) [rt=2450]
BusFMS -> FMSSrv: P1(V2) [rt=2550]
clients -> FMSSrv: C2 addpgn(P1,ES)
note left clients: Makes no difference as this is already subscribed with same type.
BusFMS -> FMSSrv: P1(V2) [rt=2650]
BusFMS -> FMSSrv: P2(V1) [rt=2725]

...
BusFMS -> FMSSrv: P1(V2) [rt=3150]
hnote over clients: dt 952ms
FMSSrv -> clients: FMS<rt=3202, P1(V2, rt=3150), P2(V1, rt=2725), P3(no-value, rt=0)>
note left clients: Here P1, P2, and P3 updates are sent at the same time. This is **not** required. 

BusFMS -> FMSSrv: P3(V1) [rt=3350]
BusFMS -> FMSSrv: P1(V2) [rt=3350]
BusFMS -> FMSSrv: P1(V3) [rt=3450]
...
BusFMS -> FMSSrv: P2(V1) [rt=3725]
...
BusFMS -> FMSSrv: P1(V3) [rt=4150]
hnote over clients: dt 972ms
FMSSrv -> clients: FMS<rt=4222, P1(V3, rt=4150), P2(V1, rt=3725), P3(V1, rt=3350)>
...

@enduml
```


```plantuml
@startuml 
title Sending_multiple_values_of_one_PGN
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
BusFMS -> FMSSrv: P1(V1) [rt=7565]
BusFMS -> FMSSrv: P1(V2) [rt=7585]
...
BusFMS -> FMSSrv: P1(V7) [rt=9625]
clients -> FMSSrv: C1 addpgn(P1,OC)
BusFMS -> FMSSrv: P1(V7) [rt=9645]
FMSSrv -> clients: FMS<rt=9650, P1(V7, rt=9645)>
BusFMS -> FMSSrv: P1(V7) [rt=9665]
BusFMS -> FMSSrv: P1(V8) [rt=9685]
BusFMS -> FMSSrv: P1(V9) [rt=9705]
BusFMS -> FMSSrv: P1(V9) [rt=9725]
BusFMS -> FMSSrv: P1(V10) [rt=9745]

FMSSrv -> clients: FMS<rt=9750, P1(V8, rt=9685), P1(V9, rt=9725), P1(V10, rt=9745)>

BusFMS -> FMSSrv: P1(V11) [rt=9765]
BusFMS -> FMSSrv: P1(V12) [rt=9785]
BusFMS -> FMSSrv: P1(V11) [rt=9805]
BusFMS -> FMSSrv: P1(V12) [rt=9825]
BusFMS -> FMSSrv: P1(V12) [rt=9845]

FMSSrv -> clients: FMS<rt=9850, P1(V11, rt=9765), P1(V12, rt=9785), P1(V11, rt=9805), P1(V12, rt=9845)>
note left clients: Values V11 and V12 are sent more than once. This is needed \nsince all values *changes* are required

...

@enduml
```


```plantuml
@startuml 
title A_bit_of_everything
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
BusFMS -> FMSSrv: P1(V1) [rt=3565]
BusFMS -> FMSSrv: P2(V1) [rt=4590]
BusFMS -> FMSSrv: P3(V1) [rt=3540]
BusFMS -> FMSSrv: P4(V1) [rt=3580]
BusFMS -> FMSSrv: P5(V1) [rt=3630]
note right BusFMS: To reflect the expected situation Frames/PGNs should be sent/received at consistent \n20/50/100/1000/10 000ms intervals. This have been relaxed for this diagram, nor are \nall Frames/PGNs on FMS CAN shown.  
...
clients -> FMSSrv: C1 addpgn(P1,OC)
BusFMS -> FMSSrv: P1(V2) [rt=3565]
FMSSrv -> clients: FMS<rt=3650, P1(V2, rt=3565)>
note right FMSSrv: Sent since P1 value change to V2
...
BusFMS -> FMSSrv: P1(V2) [rt=4465]
FMSSrv -> clients: FMS<rt=4600, P1(V2, rt=4465)>
note right FMSSrv: Sent since there is 1s since last send of P1
...
clients -> FMSSrv: C2 addpgn(P2,ES)
...
BusFMS -> FMSSrv: P2(V1) [rt=4800]
FMSSrv -> clients: FMS<rt=5150, P2(V1, rt=4800)>
note right FMSSrv: Sent due to P2 ES subscription
...
BusFMS -> FMSSrv: P1(V2) [rt=5465]
FMSSrv -> clients: FMS<rt=5600, P1(V2, rt=5465)>
note right FMSSrv: Sent since there is 1s since last P1 send
...
BusFMS -> FMSSrv: P1(V1) [rt=5765]
FMSSrv -> clients: FMS<rt=5850, P1(V2, rt=5765)>
note right FMSSrv: Sent since P1 value change to V1
...
BusFMS -> FMSSrv: P2(V5) [rt=5963]
...
FMSSrv -> clients: FMS<rt=6120, P2(V5, rt=5963)>
note right FMSSrv: Sent due to P2 ES subscription
BusFMS -> FMSSrv: P3(V6) [rt=6150]
... 
clients -> FMSSrv: C3 addpgn(P3,OC)
BusFMS -> FMSSrv: P3(V6) [rt=6330]
BusFMS -> FMSSrv: P3(V7) [rt=6380]
FMSSrv -> clients: FMS<rt=6470, P3(V7, rt=6380)>
note right FMSSrv: P3(V7) sent due to value change. \nWould be within spec to include P3(V6).
...
BusFMS -> FMSSrv: P3(V8) [rt=6580]
BusFMS -> FMSSrv: P1(V3) [rt=6650]
FMSSrv -> clients: FMS<rt=6560, P3(V8, rt=6580), P1(V3, rt=6650)>
...

@enduml
```



```plantuml
@startuml 
title Lots_of_data
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
clients -> FMSSrv: C1 addpgn(P1,OC;P2,OC;...P9,OC;P10,OC)
clients -> FMSSrv: C2 addpgn(P8,OC;P9,OC;...P19,OC;P20,OC)
...
FMSSrv -> clients: FMS<rt=3650, P1(V1),P2(V1),P3(V1),...,P20(V1)>
BusFMS -> FMSSrv: P1(V2) [rt=3652]
BusFMS -> FMSSrv: P2(V2) [rt=3654]
...
BusFMS -> FMSSrv: P10(V2) [rt=3670]
Note right of FMSSrv: Only 20ms since last send, but there is now 1100+ bytes of data pending. \nIt needs to be sent of so that ethernet frame stays below 1280. 
FMSSrv -> clients: FMS<rt=3670, P1(V2),P2(V2),P3(V2),...,P10(V2)>
BusFMS -> FMSSrv: P11(V2) [rt=3672]
BusFMS -> FMSSrv: P12(V2) [rt=3674]
...
BusFMS -> FMSSrv: P20(V2) [rt=3690]
Note right of FMSSrv: Again only 20ms since last send, but there is now 1100+ bytes of data pending. \nIt needs to be sent of so that ethernet frame stays below 1280. 
FMSSrv -> clients: FMS<rt=3690, P11(V2),P12(V2),P13(V2),...,P20(V2)>
BusFMS -> FMSSrv: P1(V3) [rt=3692]
BusFMS -> FMSSrv: P2(V2) [rt=3694]
...
BusFMS -> FMSSrv: P10(V3) [rt=3710]
BusFMS -> FMSSrv: P11(V3) [rt=3712]
BusFMS -> FMSSrv: P12(V3) [rt=3714]
...
BusFMS -> FMSSrv: P19(V3) [rt=3728]
Note right of FMSSrv: Fewer changes, so 38ms since last send, before buffer is full. 
FMSSrv -> clients: FMS<rt=3729, P1(V3),P10(V3),P11(V3),...,P19(V3)>
...

@enduml
```

```plantuml
@startuml 
title Multiple_Source_Addresses
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
clients -> FMSSrv: C1 addpgn(P1,ES)
...
BusFMS -> FMSSrv: P1(SA=B4, V2) [rt=11500]
FMSSrv -> clients: FMS<rt=11510, P1(SA=B4, V2)>
BusFMS -> FMSSrv: P2(SA=5F, V2) [rt=11780]
FMSSrv -> clients: FMS<rt=11800, P1(SA=5F, V2)>
Note right of FMSSrv: With different SA, so sent independently
...
FMSSrv -> clients: FMS<rt=12560, P1(SA=B4, V2)>
Note right of FMSSrv: Periodic update of P1(SA=B4)
FMSSrv -> clients: FMS<rt=12850, P1(SA=5F, V2)>
Note right of FMSSrv: Periodic update of P1(SA=5F)
Note right of FMSSrv: With different SA, so sent independently
...
@enduml
```

```plantuml
@startuml 
title Example_implementation_-_Periodic_Check
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
...
BusFMS -> FMSSrv: P3(V6) [rt=6330]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P4(V3) [rt=6530]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P6(V6) [rt=6590]
FMSSrv -> FMSSrv: PGNReceived()

clients -> FMSSrv: C1 addpgn(P3,OC)
clients -> FMSSrv: C1 addpgn(P4,ES)
clients -> FMSSrv: C1 addpgn(P6,OC)

FMSSrv -> FMSSrv: PeriodicCheck() # every 30ms \nTriggers Send() since last send > 950ms

FMSSrv -> clients: Send(): FMS<rt=6720, P3(V6, rt=6330), P4(V3, rt=6530), P6(V6, rt=6590)>
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P4(V4) [rt=7130]
FMSSrv -> FMSSrv: PGNReceived()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck() \n Triggers Send() since last-sent rt=6720 > 950ms ago
FMSSrv -> clients: Send(): FMS<rt=7680, P3(V6, rt=6330), P4(V4, rt=7130), P6(V6, rt=6590)>
... 
BusFMS -> FMSSrv: P3(V8) [rt=7980]
FMSSrv -> FMSSrv: PGNReceived()  # Added P3(V8) to send-buffer due to value change
FMSSrv -> FMSSrv: PeriodicCheck() # P3(V8) [rt=7980] < 70ms old
BusFMS -> FMSSrv: P3(V7) [rt=8030]
note over FMSSrv: From here on not showing PGNReceived() call
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3(V8) [rt=7980] > 70ms old \n P4 & P6 last-send < 750ms, so not addded
FMSSrv -> clients: Send(): FMS<rt=8050, P3(V8, rt=7980), P3(V7, rt=8030)>
... 
BusFMS -> FMSSrv: P3(V7) [rt=8600]
BusFMS -> FMSSrv: P4(V12) [rt=8610]
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P6(V6) [rt=8630]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P4 (and P6) last-sent > 950ms
FMSSrv -> clients: Send(): FMS<rt=8650, P4(V12, rt=8610), P6(V6, rt=8630)>
note right of FMSSrv: P3 NOT sent, as that was last-sent rt=8050, which is < 750ms
BusFMS -> FMSSrv: P3(V7) [rt=8700]
FMSSrv -> FMSSrv: PeriodicCheck()
...
BusFMS -> FMSSrv: P3(V7) [rt=8980]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3 last-send > 950ms
FMSSrv -> clients: Send(): FMS<rt=9005, P3(V7, rt=8980)>
FMSSrv -> FMSSrv: PeriodicCheck()
...
@enduml
```



```plantuml
@startuml 
title Example_implementation_-_send_early
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
...
BusFMS -> FMSSrv: P3(V6) [rt=6330]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P4(V3) [rt=6530]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P6(V6) [rt=6590]
FMSSrv -> FMSSrv: PGNReceived()

clients -> FMSSrv: C1 addpgn(P3,OC)
clients -> FMSSrv: C1 addpgn(P4,ES)
clients -> FMSSrv: C1 addpgn(P6,OC)

FMSSrv -> FMSSrv: PeriodicCheck() # every 30ms \n

FMSSrv -> clients: Send(): FMS<rt=6720, P3(V6, rt=6330), P4(V3, rt=6530), P6(V6, rt=6590)>
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P4(V4) [rt=7130]
FMSSrv -> FMSSrv: PGNReceived()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck() \n Triggers Send() since last-sent rt=6720 > 975ms ago
FMSSrv -> clients: Send(): FMS<rt=7700, P3(V6, rt=6330), P4(V4, rt=7130), P6(V6, rt=6590)>
... 
BusFMS -> FMSSrv: P3(V8) [rt=7980]
FMSSrv -> FMSSrv: PGNReceived()  # Added P3(V8) to send-buffer due to value change
FMSSrv -> clients: Send(): FMS<rt=7980, P3(V8, rt=7980)>
note right of FMSSrv: More than 50ms since last send, send new value at once
FMSSrv -> FMSSrv: PeriodicCheck() 
BusFMS -> FMSSrv: P3(V7) [rt=8030]
FMSSrv -> FMSSrv: PGNReceived()  # Added P3(V7)  to send-buffer due to value change
note right of FMSSrv: Less than 50ms since last send, so wait with send
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3(V7) in send-buffer
FMSSrv -> clients: Send(): FMS<rt=8050, P3(V7, rt=8030)>
... 
BusFMS -> FMSSrv: P3(V7) [rt=8600]
BusFMS -> FMSSrv: P4(V12) [rt=8610]
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P6(V6) [rt=8630]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P4 (and P6) last-sent > 975ms
FMSSrv -> clients: Send(): FMS<rt=8680, P4(V12, rt=8610), P6(V6, rt=8630)>
note right of FMSSrv: P3 NOT sent, as that was last-sent rt=8050, which is < 750ms ago
BusFMS -> FMSSrv: P3(V7) [rt=8700]
FMSSrv -> FMSSrv: PeriodicCheck()
...
BusFMS -> FMSSrv: P3(V7) [rt=8980]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3 last-send > 950ms
FMSSrv -> clients: Send(): FMS<rt=9005, P3(V7, rt=8980)>
FMSSrv -> FMSSrv: PeriodicCheck()
...
@enduml
```



Diagrams still here, as they are source!

```plantuml
@startuml 
title BAM overview
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
...

clients -> FMSSrv: addpgn(PGN_ID=F345,ES)
BusFMS -> FMSSrv: PGN(ID=F345, data="ABCDEF") [rt=2200]
note over FMSSrv: For PGN=F345 save "ABCDEF" as value
FMSSrv -> clients: FMS<rt=2222, PGN(ID=F345, data "ABCDEF", rt=2200)>
FMSSrv -> clients: FMS<rt=3221, PGN(ID=F345, data "ABCDEF", rt=2200)>
note over FMSSrv: PGN F345 initial sent as one frame (< 8 bytes)
BusFMS -> FMSSrv: PGN(ID=TP.CM (ECFF), size=19, packets=3, PGN=F345 ) [rt=3700]
FMSSrv -> FMSSrv: BAM transmission detected, enter BAM state-machine
... 50 - 200ms ...
BusFMS -> FMSSrv: PGN(ID=TP.DT (EBFF), seq_nbr=1, data="ABCDEFG") [rt=3810]
... 50 - 200ms ...
BusFMS -> FMSSrv: PGN(ID=TP.DT (EBFF), seq_nbr=2, data="HIJKLMN") [rt=3905]
... 50 - 200ms ...
BusFMS -> FMSSrv: PGN(ID=TP.DT (EBFF), seq_nbr=3, data="OPQRS") [rt=4015]
FMSSrv -> FMSSrv: BAM transmission complete, save "ABCDEFGHIJKLMNOPQRS" as data for PGN F345
...
FMSSrv -> clients: FMS<rt=4190, PGN(ID=F345, data "ABCDEFGHIJKLMNOPQRS", rt=4015)>
note right of FMSSrv: Timestamp for last TP.DT frame used for PGN.
...
BusFMS -> FMSSrv: PGN(ID=F345, data="XQYZ") [rt=13000]
note right BusFMS: PGN have become 8 bytes or less and "moved back" to single frame transmission
FMSSrv -> clients: FMS<rt=13200, PGN(ID=F345, data "XQYZ", rt=13000)>
... 
@enduml
```

Diagram of "old" BAM data being sent halfway trough transmission of new BAM data:

```plantuml
@startuml 
title BAM overview (valid data)
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
...

clients -> FMSSrv: addpgn(PGN_ID=F345,ES)
...
FMSSrv -> clients: FMS<rt=2950, PGN(ID=12345, data "ABCDEF", rt=2200)>
note over FMSSrv: PGN F345 initial sent as one frame (< 8 bytes)
BusFMS -> FMSSrv: PGN(ID=TP.CM (ECFF), size=19, packets=3, PGN=F345 ) [rt=3700]
FMSSrv -> FMSSrv: BAM transmission detected, enter BAM state-machine

BusFMS -> FMSSrv: PGN(ID=TP.DT (EBFF), seq_nbr=1, data="ABCDEFG") [rt=3810]

BusFMS -> FMSSrv: PGN(ID=TP.DT (EBFF), seq_nbr=2, data="HIJKLMN") [rt=3905]

note over FMSSrv: It is time to send PGN=F345 - use current valid data, **not** partially received.
FMSSrv -> clients: FMS<rt=3920, PGN(ID=12345, data "ABCDEF", rt=2200)>

BusFMS -> FMSSrv: PGN(ID=TP.DT (EBFF), seq_nbr=3, data="OPQRS") [rt=4015]
FMSSrv -> FMSSrv: BAM transmission complete, save "ABCDEFGHIJKLMNOPQRS" as data for PGN 12345
...
FMSSrv -> clients: FMS<rt=4915, PGN(ID=F345, data "ABCDEFGHIJKLMNOPQRS", rt=4015)>
...
@enduml
```

```plantuml
@startuml 
title Example_implementation_-_Periodic_Check
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
...
BusFMS -> FMSSrv: P3(V6) [rt=6330]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P4(V3) [rt=6530]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P6(V6) [rt=6590]
FMSSrv -> FMSSrv: PGNReceived()

clients -> FMSSrv: C1 addpgn(P3,OC)
clients -> FMSSrv: C1 addpgn(P4,ES)
clients -> FMSSrv: C1 addpgn(P6,OC)

FMSSrv -> FMSSrv: PeriodicCheck() # every 30ms \nTriggers Send() since last send > 950ms

FMSSrv -> clients: Send(): FMS<rt=6720, P3(V6, rt=6330), P4(V3, rt=6530), P6(V6, rt=6590)>
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P4(V4) [rt=7130]
FMSSrv -> FMSSrv: PGNReceived()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck() \n Triggers Send() since last-sent rt=6720 > 950ms ago
FMSSrv -> clients: Send(): FMS<rt=7680, P3(V6, rt=6330), P4(V4, rt=7130), P6(V6, rt=6590)>
... 
BusFMS -> FMSSrv: P3(V8) [rt=7980]
FMSSrv -> FMSSrv: PGNReceived()  # Added P3(V8) to send-buffer due to value change
FMSSrv -> FMSSrv: PeriodicCheck() # P3(V8) [rt=7980] < 70ms old
BusFMS -> FMSSrv: P3(V7) [rt=8030]
note over FMSSrv: From here on not showing PGNReceived() call
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3(V8) [rt=7980] > 70ms old \n P4 & P6 last-send < 750ms, so not addded
FMSSrv -> clients: Send(): FMS<rt=8050, P3(V8, rt=7980), P3(V7, rt=8030)>
... 
BusFMS -> FMSSrv: P3(V7) [rt=8600]
BusFMS -> FMSSrv: P4(V12) [rt=8610]
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P6(V6) [rt=8630]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P4 (and P6) last-sent > 950ms
FMSSrv -> clients: Send(): FMS<rt=8650, P4(V12, rt=8610), P6(V6, rt=8630)>
note right of FMSSrv: P3 NOT sent, as that was last-sent rt=8050, which is < 750ms
BusFMS -> FMSSrv: P3(V7) [rt=8700]
FMSSrv -> FMSSrv: PeriodicCheck()
...
BusFMS -> FMSSrv: P3(V7) [rt=8980]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3 last-send > 950ms
FMSSrv -> clients: Send(): FMS<rt=9005, P3(V7, rt=8980)>
FMSSrv -> FMSSrv: PeriodicCheck()
...
@enduml
```

```plantuml
@startuml 
title Example_implementation_-_Send_Early
autonumber "<b>[000]"
participant BusFMS as "FMS CAN"
participant FMSSrv as "FMStoIP\nserivce"
collections clients
...
BusFMS -> FMSSrv: P3(V6) [rt=6330]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P4(V3) [rt=6530]
FMSSrv -> FMSSrv: PGNReceived()
BusFMS -> FMSSrv: P6(V6) [rt=6590]
FMSSrv -> FMSSrv: PGNReceived()

clients -> FMSSrv: C1 addpgn(P3,OC)
clients -> FMSSrv: C1 addpgn(P4,ES)
clients -> FMSSrv: C1 addpgn(P6,OC)

FMSSrv -> FMSSrv: PeriodicCheck() # every 30ms \n

FMSSrv -> clients: Send(): FMS<rt=6720, P3(V6, rt=6330), P4(V3, rt=6530), P6(V6, rt=6590)>
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P4(V4) [rt=7130]
FMSSrv -> FMSSrv: PGNReceived()
FMSSrv -> FMSSrv: PeriodicCheck()
...
FMSSrv -> FMSSrv: PeriodicCheck() \n Triggers Send() since last-sent rt=6720 > 975ms ago
FMSSrv -> clients: Send(): FMS<rt=7700, P3(V6, rt=6330), P4(V4, rt=7130), P6(V6, rt=6590)>
... 
BusFMS -> FMSSrv: P3(V8) [rt=7980]
FMSSrv -> FMSSrv: PGNReceived()  # Added P3(V8) to send-buffer due to value change
FMSSrv -> clients: Send(): FMS<rt=7980, P3(V8, rt=7980)>
note right of FMSSrv: More than 50ms since last send, send new value at once
FMSSrv -> FMSSrv: PeriodicCheck() 
BusFMS -> FMSSrv: P3(V7) [rt=8030]
FMSSrv -> FMSSrv: PGNReceived()  # Added P3(V7)  to send-buffer due to value change
note right of FMSSrv: Less than 50ms since last send, so wait with send
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3(V7) in send-buffer
FMSSrv -> clients: Send(): FMS<rt=8050, P3(V7, rt=8030)>
... 
BusFMS -> FMSSrv: P3(V7) [rt=8600]
BusFMS -> FMSSrv: P4(V12) [rt=8610]
FMSSrv -> FMSSrv: PeriodicCheck()
BusFMS -> FMSSrv: P6(V6) [rt=8630]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P4 (and P6) last-sent > 975ms
FMSSrv -> clients: Send(): FMS<rt=8680, P4(V12, rt=8610), P6(V6, rt=8630)>
note right of FMSSrv: P3 NOT sent, as that was last-sent rt=8050, which is < 750ms ago
BusFMS -> FMSSrv: P3(V7) [rt=8700]
FMSSrv -> FMSSrv: PeriodicCheck()
...
BusFMS -> FMSSrv: P3(V7) [rt=8980]
FMSSrv -> FMSSrv: PeriodicCheck()
FMSSrv -> FMSSrv: PeriodicCheck() # Triggers Send() as P3 last-send > 950ms
FMSSrv -> clients: Send(): FMS<rt=9005, P3(V7, rt=8980)>
FMSSrv -> FMSSrv: PeriodicCheck()
...
@enduml
```

