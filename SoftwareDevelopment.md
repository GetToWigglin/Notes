# Software Development Notes
## Glossary
https – hyper text transport protocol secure, this is a form of http communication in which the communications are encrypted on transit and decrypted at the destination. This is achieved by SSL and more recently Transport Layer Security (TLS). 
Symmetric Key Encryption – The same key is used to encrypt and decrypt information. It’s faster, more compact, and more efficient, but it only provides confidentiality.
Asymmetric Key Encryption – Different keys are used for encryption and decryption. More complicated and more overhead but allows for authentication and non-repudiation as well as confidentiality.

## Encryption
The Process
1.	Server and Client agree on the version of protocol to use
2.	Select cryptographic algorithms
3.	Authenticate each other by exchanging and validating digital certificates
4.	Utilize asymmetric encryption to exchange a shared secret key and then communicate symmetrically encrypted using that key
More in depth
https://www.ibm.com/docs/en/ibm-mq/9.1?topic=tls-overview-ssltls-handshake
1.	Client says hello, provides cryptographic information it has available, as well as a random byte string (used later)
2.	Server responds, providing a digital certificate. If the server requires a certificate, then it will provide the types of accepted certificates
3.	Client verifies the certificate
4.	Client sends the random byte string encrypted using the public key which will be used to generate the shared symmetric key
5.	If the server sent a “client certification request” the server will also perform steps 3 and 4 if that fails or no certification is required it will respond with a “no digital certificate alert”
6.	The client sends a finished message encrypted with the secret symmetric key
7.	The server sends a finished message encrypted with the secret symmetric key
8.	Communication is now exchanged via symmetric encryption with the secret symmetric key that was generated from the random byte string
How is the symmetric key exchanged? Two ways of exchanging: Key Encipherment or Key Agreement (Diffie-Hellman algorithm), these two are mutually exclusive. 
https://security.stackexchange.com/questions/130938/how-does-the-symmetric-key-get-exchanged-in-ssl-tls-handshake
Discrepancy between internal updating and external updating. Seems like they want the external updates immediately, but configuration is possible. The event recorder should be primarily a passthrough.

## Modern Web Architecture
Dependency Inversion – This concept has the establishment of interfaces for abstraction such that instead of having parents referencing children, the children and parents both reference common interfaces, thus keeping the classes decoupled. This results in a sort of “inverted” compile time dependency, while still maintaining the linear direction of run-time execution.
The design of Https is that the server (listener/responder) hosts the certificate and sends it to the client (requester).

Windows + shift + s – Screen shot snipper tool
Debt to Income Ratio – Total Income / Total Debt
In order for AMD and AMDServices to communicate, they need to be using the same ports for communication.
1.	Right click AmdOrderEditor.Api and select properties
2.	Debug -> Open debug launch profiles UI
3.	URLs are set at the bottom of the window
findstr /c:"StageBurnOutDetected" *.json

Aggregate Initialization – If you use aggregate initialization (TestStruct X = { 0… }) The members will be initialized in order and any not explicitly initialized will use the empty initializer list equivalent (x{} is x with an empty initializer list). An empty initializer list performs value initialization (x() or new x() is value initialization).
Basically what this statement is saying is "Initialize the first member of X to 0, then use the empty initializer list for all the other elements (Basically if we did X = {} or X{}). Furthermore, the empty initializer list performs value initialization. There are some instances where value initialization cannot be performed (such as references) so the syntax X = {0} is unnecessarily convoluted for default initializing an array.

So what happened was the winapi CreateThread has a max limit based on the default stack size the OS will allocate to individual threads, this is essentially 2048 without reducing the default stack size. I couldn’t find anything saying exactly what happens when you exceed that. It seems like the program should crash, that’s what I would expect I think. Instead, it seems like it MAY be queuing the tasks up and getting to the threads when it can. I ran with AMD Services and I was getting the execution exactly as I suspected. So it seems to me, that the delay in closing threads caused by the timeout allows the number of threads to stack up and causes the issue.

## Computer Graphics
Sampling (Signal Processing) – This is the reduction of a continuous-time signal to a discrete-time signal (Analog to Digital). A sample is a value of a signal at a point in time and/or space, basically a value at time t of a signal function. An ideal sampler would be able to perfectly produce samples equal to the instantaneous value of the continuous signal function.
Distortion – Deviations in the digitally constructed signal to the analog signal.
Nyquist Rate – A property of a continuous-time signal. If an analog signal cycles at a rate of 10 cycles per second, the Nyquist Rate would be 10 X 2, or 20 cycles/second. 
Nyquist Frequency – A property of a discrete-time signal. If a digital signal sampling rate of 10 samples per second, the Nyquist Frequency would be 10 X 0.5, or 20 cycles/sample.
How do these two values work together in sampling and what do they mean? Sampling is the process of turning a continuous function into a discrete function. Essentially, measurements of the continuous function are taken at regular intervals to “compress” into a digital function. This discrete function can then be “unpacked” via interpolation to mimic the original continuous function. In regular words, if we are recording a music performance we will be taking snapshots of the sound at regular intervals. The more frequent the sampling interval the more correct the result will be when we play the sound back at a given time that may not correspond to a sampled instance. The reason this is important is because there is a specific sampling frequency that must be used for a given frequency range of a continuous function to prevent distortion known as “aliasing”. This is described by the Nyquist-Shannon Sampling Theorem. In other words, this theorem establishes the sample rate necessary to capture “all” the information from a continuous-signal of finite bandwidth (frequency).
Aliasing is the overlapping of 
