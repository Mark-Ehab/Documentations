# <b><u>Content:</u></b>



# 1. <b><u>intro:</u></b>

![[CAN_network.png]]
***COM stack*** is used to communicate between the ECUs so, lets consider the following scenarios:
1. the <b><u><i>Master</i></u></b> ECU is sending a ***==periodic==*** message that contains the following ***==signals==***: kilometers, Speedand time stamp to all the other ECUs where these ECUs use that data when taking a snapshot if any problem happens, so, for that we need the COM stack.
2. <b><u><i>ECU_1</i></u></b> sends this data that contains the gear state to <b><u><i>ECU_2</i></u></b> and according to the gear state if it's ***R (Reverse state)*** the camera should be functioning to let the user see what is behind the car.
there are more scenarios of course than those 2, so, COM stack is one of the most important stacks in the CAR.
there are more than one type of COM stack as follows:
1. COM stack based on ***==CAN==***.
2. COM stack based on ***==Ethernet==***.
3. COM stack based on ***==LIN==***.
4. COM stack based on ***==flexRay==***.
but we will focus on the <b><u><i>COM stack based on CAN</i></u></b>.
***==Note:==*** the requirements of the communication stack is prepared by the <b><i><u>OEM</u></i></b> in the form of ***communication matrix (CAN DB - CAN data base)*** where this matrix contains number of messages, which message is sent by which ECU, how many signals are there in each message, the size of these signals and so on.
***==Note:==*** there are tools for requirement named ***requirement management tools*** and one of the famous tools <b><i><u>Doors</u></i></b>.

![[Car_networks_example.png]]
more than one communication protocol can be in the same ECU and this ECU can be used as gateway between the ECUs that are communicating through different protocols and hence this ECU is named ***==Gateway ECU==***.
according to the pic. above the used ***Gateway ECU*** has 3 communication protocols which are ***CAN***, ***Ethernet***, ***LIN***, to be able to translate between those in the CAN network, LIN network and Ethernet network.
# 2. <b><u>AUTOSAR layered architecture:</u></b>

![[COM_stack_architecture_AUTOSAR.png]]
this diagram shows the whole architecture including more than communication protocol module as ***FlexRay***, ***CAN***, ***LIN***, but we will focus on the ***==CAN==***.

![[COM_stack_architecture_based_on_CAN.png]]
in this diagram the COM stack architecture based on ***CAN*** where we can split the modules into 2 groups:
1. ***Network control path:*** where the communication path is opened or closed, so, it is                                                         represented as the muscles.
2. ***Data path:*** where the logic of transporting the data whether collecting them through Com                          module, routing the PDUs through the PduR and so on (the functionality of the                          Com module and the PduR will be covered later).


# 3. <b><u>COM stack concepts and terminologies:</u></b>

1. <b><u><i>SDU</i></u></b>: Service Data Unit (raw data)
2. <b><u><i>PCI</i></u></b>: Protocol Control Information (some data relative to the protocol itself)
3. <b><u><i>UD</i></u></b>: User Data.
4. <b><u><i>PDU</i></u></b>: Packet Data Unit
        `PDU = PCI + UD + SDU`.
	
	![[PDU_PCI_SDU.png]]
	***==Note==***: every ***PDU*** is transported from layer to the layer below as ***SDU*** where some ***PCI*** is                    added to this ***SDU*** to form a new ***PDU*** that will be transported again to the next layer              where it will be treated as ***SDU*** again and so on.
5. <b><u><i>Signals</i></u></b>: Signals are the smallest entities for the exchange of information.
6. <b><u><i>Signal group</i></u></b>: logical grouping of some signals (i.e., if there is more than one signals that                                 should be sent at the same time or we can say that the receiver will need them                           all at once as the environment data of the car like the speed and so on, so, these                        signals are grouped together in one signal group).
	![[Signal_signalGroup.png]]
	as shown in this pic the SWC (Software Component) in the Application layer communicates only in signals and signal groups then send these signals and signal groups to the RTE (Run-Time Environment) where the RTE forwards these signals and signal groups to a modules named ***==Com==*** where those signals and signal groups are packed into a single PDU
	`PDU = Signals + Signal gorups` and the ***PDU*** can be consisted of signals only or signal groups only.
# 4. <b><u>Modules Roles and features:</u></b>

## <b><u>Com (Communication module):</u></b>
### 1. <b><i><u>Role:</u></i></b>

### 2. <b><i><u>Features:</u></i></b>
#### 1. <b><u><i>Communication mode:</i></u></b>
##### a. <b><u><i>Transmission mode (of the PDU):</i></u></b>
- <u><i><b>Direct</b></u></i>: ***Application*** is the one which takes the decision to send the PDU (incase the PDU is                     configured to be Direct), Where the ***APP*** tells the ***Com module*** through ***RTE*** to send this             PDU.
		![[Direct_transmission_mode.png]]
		***==Note==***: sending a **PDU** configured to be in ***Direct mode*** will rely on the <b><u><i>transfer property</i></u></b>             of its signals.
		
- <u><i><b>Periodic</b></u></i>: Com module is responsible of sending the PDU periodically (incase the PDU is                             configured to be periodic), where the Com module has its own timers to calculate this                time in order to send this PDU periodically.
		![[Periodic_transmission_mode.png]]
		
- <u><i><b>Mixed</b></u></i>: it is a mix of the 2 modes above where the ***Com module*** sends the PDU every                              configured period and also the ***App*** through the ***RTE*** tells the ***Com module*** to send this              PDU at a certain time, so, the ***Com module*** is responsible of sending this PDU in the                    configured periods and also if the ***App*** wants to send it in any time other than the                       period that is configured in the ***Com module*** it will contact the ***Com module*** through                  the ***RTE*** and tells it to send the PDU. 
		![[Mixed_transmission_mode.png]]
##### b. <b><u><i>Transfer property (of the signal):</i></u></b>
- <u><i><b>Triggered</b></u></i>: if a signal is configured as triggered then this means that if there is a **PDU** that has                    this signal that is configured as ***triggered*** and the ***App*** sends this signal to the                              ***Com module*** then the whole ***PDU*** will be transmitted from the ***Com module*** to the                     ***PduR*** ==immediately== regardless of the other signals in that ***PDU***.
- <u><i><b>Triggered on change</b></u></i>: if a signal is configured as triggered-on-change signal this means that if                                       there is a ***PDU*** that has this signal that is configured as ***triggered-on-                                           change*** and the ***App*** sends this signal to the ***Com module*** then the                                               whole ***PDU*** will be transmitted from the ***Com module*** to the ***PduR*** iff ***==the==                                      ==value of this signal has changed from the last time it was sent==***.
	an example on <b><u><i>triggered</i></u></b> and <b><u><i>triggered-on-change</i></u></b>:
	
	![[triggered_triggered_on_change_signals.png]]
	if we have a **PDU** as shown in the pic. above where this **PDU** consists of 3 signals: 
		- <b><u><i>Signal #1</i></u></b>: triggered signal.
		- <b><u><i>Signal #2</i></u></b>: triggered signal.
		- <b><u><i>Signal #3</i></u></b>: triggered-on-change signal.
	so, lets consider the following 4 scenarios:
	<b><u>First:</u></b>
	![[sending_triggered_signal_1.png]]
	as shown in this pic. ***==Signal #1==*** is been sent from ***APP_1*** to the ***Com module*** through the ***RTE***  and since ***==Signal #1==*** is ***==triggered signal==*** then this ***PDU*** should be sent immediately from the ***Com module*** to the ***PduR*** regardless the value of this signal is been changed from the last time it was sent at and regardless of whether the other signals are sent or not.
	<b><u>Second:</u></b>
	![[Sending_triggered_signal_2.png]]
	as shown in this pic. ***==Signal #2==*** is been sent from ***APP_1*** to the ***Com module*** through the ***RTE***  and since ***==Signal #2==*** is ***==triggered signal==*** then this ***PDU*** should be sent immediately from the ***Com module*** to the ***PduR*** regardless the value of this signal is been changed from the last time it was sent at and regardless of whether the other signals are sent or not.
	<b><u>Third:</u></b>
	![[Sending_triggered_on_change_signal_changed.png]]
	as shown in this pic. ***==Signal #3==*** is been sent from ***APP_1*** to the ***Com module*** through the ***RTE***  and since ***==Signal #3==*** is ***==triggered-on-change signal and its value is being changed from the last time it was sent (i.e., its value now equal 10 and its previous value was 0 this means its value changed)==*** then this ***PDU*** should be sent immediately from the ***Com module*** to the ***PduR*** regardless of whether the other signals are sent or not.
	<b><u>Forth:</u></b>
	![[Sending_triggered_on_change_signal_unchanged.png]]
	as shown in this pic. ***==Signal #3==*** is been sent from ***APP_1*** to the ***Com module*** through the ***RTE***  and since ***==Signal #3==*** is ***==triggered-on-change signal and its value isn't been changed from the last time it was sent (i.e., its value now equal 10 and its previous value was also 10 this means its value is the same)==*** then this ***PDU*** <b><i><u>shouldn't</u></i></b> be sent from the ***Com module*** to the ***PduR*** and this ***PDU*** should <b><i><u>wait</u></i></b> till one of the other triggered signals (i.e., signal #1 or signal #2) is sent or this triggered-on-change signal (i.e., signal #3) is sent again but with different value.
- <u><i><b>Pending</b></u></i>:

# 5. <b><u>Full pic. from the app to the physical bus (Sending):</u></b>

![[Sending_PDU_BlockDiagram.png]]


# 6. <b><u>Full pic. from the physical bus to the App (Receiving):</u></b>


# 6. <b><u>Sequence diagrams:</u></b>
