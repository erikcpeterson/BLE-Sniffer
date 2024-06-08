To test this project, open Xcode and select Clone Git Repository, and enter this URL. To run, you will need two iOS devices (I tested with an iPhone 8 and an iPhone 13, an iPad should, but a Macbook won't work, neither will running it on a simulator as it needs the bluetooth low energy capability). On one of the devices adjust your port if you want (or leave it to the default of 180D). Then click on the giant blue button, this will effectively start sending a SYN message, and will wait for a device to respond back. On your other device open up, change the port to the same one used on the first device and go to the receive data tab (which acts as a server). This device should now receive the SYN message and send a SYN+ACK message back, wait until it receives an ACK message and then will receive the message. 

Originally I started out by trying to create a project that was designed to exploit Apple's bluetooth low energy exploit, which allows for a lot of information to be captured when using an iPhone. I was unable to purchase the nRF52840 dongle (as it would take multiple weeks to ship), which acts as a bluetooth low energy sniffer and can send its own packets. I tried for a few days to make do with an ASUS BT500 adapter, which has BLE capabilities, however this adapter wasn't able to easily sniff ble messages, and more importantly, I couldn't find a way to send ble packets.

After some further research, I realized that nearly all mobile phones have this capability to send and receive ble packets so I adapted my project to send and receive messages from one phone to another using bluetooth low energy, which is a good way to effectively "sneak" a message, similair to what Apple does, as ble usually contains so much junk signals (which has only worsened with IoT), so messages can pretty much be hidden in plain sight. 

For this project, I mainly focused on creating the functionality part of the app, there are some bugs which are outlined further below. Instead of making a flashy app, I tried to focus on the actual connection and three way handshake part of the app, which is visualized really well when running the app. I learned that the ble protocols that are available to use in Swift pretty much already implement a form of a three way handshake, but for an added educational benefit, I decided to add manual three way handshake with SYN, SYN+ACK, etc. messages and confirmations. The ViewController (which acts as the client) first checks if bluetooth is on, then starts advertising on the port as "SYN Message". The server (which operates on BLEClientViewController), then is able to connect to the client, checks if various characteristics match, and then sends data which is the SYN+ACK message. The client then receives the message, loads the ACK message, which the server then reads, "unlocking" the client message which the server then reads. 

Since we aren't sending heavy data like pictures, the entire data is transmitted in one packet. Because of this, the message data acts as the actual data, as well as a FIN message. When the server receives the message, it sends a FIN+ACK message, and upon receiving the FIN+ACK message, the client sends a final ACK, and immediately turns off broadcasting. The server will then display the ACK message upon receiving it and the client turns off.

For better visual display, I added some sleep statements throughout the code. This exagerates the delay between the sending and receiving of various messages helps people unfamiliar with the three way handshake method better understand how it works.

Further information:
As outlined above, the ViewController.swift acts as the client while the BLEClientViewController.swift acts as the server. I originally had them reversed (hence the name) but decided to have the ViewController.swift act as the client due to the fact that it was sending the original data to the "server". However some of the functions are ambigous, and tend to mix server and client behaviors. This is because the protocols employed by Swift to communicate via ble tend to act as a client server box, where each one can put in information and is notified when it is taken out. If I were to recreate this, I would probably have not switched the client and server (keeping the ViewController the server and the BLEClientViewController the client), however I don't think it matters a lot and would take a lot of time to rewrite the SYN and ACK messages to work the other way around. 

Bugs:
The only main bug I ecnountered happens when someone tries to send a second message without refreshing the app (this is a problem with the BLEClientViewController.swift file not being able to successfuly unsubscribe from notifications when clicking the dismiss button). I tried tinkering around with various lines to try and refresh it but ultimately wasn't able to fix it. The bug is minor as it still transmits the data, however some features like the ble broadcasting turning off upon the server receiving the final ACK message doesn't work as all of the messages become duplicated (as the server becomes double subscribed to the server). I may try and fix this in a later update.
