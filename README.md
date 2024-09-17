java c
CS5229 Advanced Computer Networks 
Programming Assignment 1 
Semester 1 AY 2024/25
1 Overview In this programming assignment, you will explore sketch-based algorithms designed to mon- itor network traffic across a high-traffic network link.  We will provide you with real-world network traces and a skeleton code structure to guide you in completing tasks related to network monitoring.The primary aim of this assignment is to help you understand the delicate balance be- tween memory usage, computational resources, and the accuracy achieved when employing sketch-based estimation algorithms in real-world network monitoring scenarios.
References: Lecture 2 - Network Monitoring
Honour Code 
This assignment constitutes 12% of your final grade for this module. You are expected to work on the assignment individually. While you may discuss concepts with your classmates and refer to online resources, plagiarism will not be tolerated. You are expected to adhere to the NUS code of conduct. 
2 Submission Instructions 
2.1 Items to be Submitted 
• Source Code (task1. py,  task2. py,  task3. py) - Your implementation for each task should be submitted after editing the corresponding files.
• Report (report-student   id. pdf) - A brief and concise report detailing your algorith- mic approach and the results of your implementations.  Please include your name and Student ID (AXXXXXXXY) in the top left corner of the first page of the report.
2.2 Submission Folder Structure and Instructions 
Organize your files into the following folder structure:
AXXXXXXXY Assignment1/ 
−Code/
−−task1 . py
−−task2 . py
−−task3 . py
−report−student   id . pdfZip the main folder (AXXXXXXXY Assignment1/) and name it using the following for- mat:  StudentID   5229p1.zip  (e.g., A1234567X 5229p1. zip).  Submit the zipped file to the Canvas folder  “Programming Assignment 1” .
Submission Deadline:  16 September 2024, 8:00am. Late submissions will incur a penalty of 20% of the total grade for each day they are late.
3    Background - Network Monitoring Network measurement is crucial for ensuring the efficient operation of networks.  To facilitate effective network management, it is essential to obtain fine-grained measurements, including metrics such asper-flow size, heavy hitter detection, and  cardinality.  These measurements play a pivotal role in various network management tasks, including load balancing, conges- tion control, quality of service assurance, scheduling optimization, and anomaly detection. Software switches have emerged as versatile platforms for conducting network measurements. However, a significant challenge persists as network link speeds now reach up to 400 Gbps, and switching capacity has exceeded tens of Tbps.  Consequently, there is a pressing need to address the scalability of software-based processing to keep pace with the ever-increasing demands of high-speed networks.One approach to network monitoring is to capture and compute aggregated statistics directly in the data plane. These aggregated data can then be further processed in the control plane for more complex tasks. In this assignment, you will gain insights into the utilization of sketches as a means of aggregating statistics in the data plane. This approach allows for efficient preliminary data analysis within the network infrastructure itself, forwarding only the relevant information to the control plane for advanced processing and decision-making when required.
The Need for Sketch-Based Techniques 
Consider the example of flow-frequency estimation, where the primary goal is to determine the frequency of each unique flow within a network trace.
• While ana¨ıve approach, such as a counter-based method, might suffice for a small num- ber of unique flows in the network, these solutions become impractical in real-world scenarios. The primary challenge arises from the fact that the memory requirements for these solutions increase linearly with the scale of data.  For instance, utilizing counters for each flow would demand several terabytes of storage, far exceeding the capabil- ities of typical commodity hardware, making such solutions unsuitable for practical implementation.
• Another consideration could be capturing and processing packets  offline.   However, given the high network link speeds, the volume of data collected would be massive. Moreover, several critical network monitoring tasks necessitate real-time monitoring to ensure efficient network operation.
• Alternatively, sampling-based techniques, which consider a subset of the entire network trace to estimate network metrics, can reduce measurement overhead.  However, it’s important to note that this method cannot provide highly accurate and fine-grained statistics.  Often, there is an inevitable loss in measurement resolution and accuracy when employing sampling-based techniques.In light of these challenges, sketch-based techniques emerge as a promising solution for ad- dressing the complexities of network monitoring, offering away to achieve accurate measure- ments and analysis while efficiently managing memory usage and computational resources.

Figure  1:  The figure illustrates flow insertion into a Count-Min Sketch, where a flow f arrives. Independent hash functions  hi (.) are used to update the sketch values. Later, for flow frequency estimation, the sketch can be queried.
Allowing Lossy Compression of Statistics:  Count-Min Sketch


In the lecture, you were introduced to essential concepts such as the Bloom Filter [2, 1] and sketches, with a particular emphasis on the Count-Min Sketch [4]. These techniques, often referred to as lossy compression methods, offer the advantage of representing network state in sub-linear space. The underlying insight driving the adoption of these techniques is that in practical applications, an exact answer isn’t always necessary. Instead, what matters is obtaining a reasonably accurate estimate.
The Count-Min Sketch, in particular, has been implemented in real-world systems [3]. For instance, at Google, a precursor to the Count-Min Sketch, known as the “count sketch,” has been implemented on their MapReduce parallel processing infrastructure.
We recommend that you refer to resources, such as this paper [4, 3], to explore error analysis and design choices when implementing a Count-Min Sketch.
The operation of a basic Count-Min Sketch is illustrated in Figure 1, showing a basic flow insertion and inference for frequency estimation.
ElasticSketch
In resource-constrained live network scenarios, it is not feasible to maintain different sketches for different tasks. As you will experience in this assignment, the Count-Min Sketch has its limitations when it comes to achieving high accuracy across multiple tasks. The main is-sue that arises is that, given a memory constraint, a Count-Min Sketch can maintain decent accuracy by using a high number of low-capacity counters (i.e., large width and depth). How-ever, elephant flows with high frequencies can cause counter overflows, leading to significant inaccuracies. ElasticSketch [5] addresses this issue by using ostracism to separate elephant flows from mouse flows. You are recommended to read the paper to better understand the concepts.


4 Setup 
Please obtain the required files from ProgrammingAssignment1 . zip.  You can install the necessary Python modules using the following command:
pip  install  -r  requirements . txt
You are not allowed to import any additional libraries for your implementation.
4.1 Input Data Live internet traffic is captured in a  .pcap file, which contains packet headers and payloads. For this assignment, essential features have been extracted from the  .pcap f代 写CS5229 Advanced Computer Networks Semester 1 AY 2024/25Python
代做程序编程语言ile into a  . csv file. You can unzip the packet trace files (packet-trace-x. csv) located in the Data/ folder. This data includes the following information:
• Packet Header Fields (X): These  fields  contain  information  from  the  packet’s header, used to identify and categorize the 5-tuple flow. The fields include:
Source   IP,  Destination   IP,  Source Port,  Destination Port, and Protocol.
• Packet Length (frame. len): This field specifies the length of the packet frame.
All packets sharing the same header fields X  are considered part of the same 5-tuple flow.
5 Assignment Objectives 
The primary objective of this assignment is to design and implement sketch algorithms capable of efficiently handling three key network monitoring tasks simultaneously.
5.1    Network Monitoring Objectives for Each Sketch 
For each sketch, your implementation must simultaneously achieve the following goals:
5.1.1 Flow Frequency Estimation Your sketch should be capable of counting the number of times a specific flow appears in the given data trace.  You are required to complete the estimate_frequency(self,  flow_X) function.
Input: A packet flow (flow_X), defined by a 5-tuple (Source IP, Destination IP, Source Port, Destination Port, and Protocol).
Output: An integer value indicating the observed frequency of the given flow.
5.1.2    Cardinality Estimation Cardinality refers to the number of unique flows observed in the given network trace.  Im- plement this functionality in the count_unique_flows() function.  Each packet flow  (flow_X) is defined by a 5-tuple (Source IP, Destination IP, Source Port, Destination Port, and Pro- tocol).
Output: An integer value indicating the number of unique flows observed.
5.1.3 Heavy Hitter Detection 
A “Heavy Hitter” in this context refers to one of the top 100 flows in terms of the number of packets observed. Implement the find_heavy_hitters() function to identify and report all heavy hitters in the packet trace.
Output: A list of the heavy hitter flows and their corresponding sizes.
In Section 5.2, for Task I and Task II, the flow is defined by the 5-tuple, as in the above monitoring tasks.
1. Source IP: To determine the origin of the attack.
2. Destination IP: To identify the target of the attack.
3. Source Port: To identify services used to launch the attack.
4. Destination Port: To identify the specific service or endpoint under attack.Therefore, only for Task III, in addition to identifying the top 100 5-tuple flows, we also require the top 20 heavy hitters for Source IP, Destination IP, Source Port, and Destination Port.The assignment involves designing and implementing sketches that can handle all three objectives—Flow Frequency Estimation, Cardinality Estimation, and Heavy Hitter Detection—using a single instance of the data structure.  This means you must create a versatile sketch that addresses various network queries without the need to instantiate a new data structure for each task.  Additionally, you must ensure efficient memory utilization. This approach mirrors real-world scenarios where a single sketch is employed to meet multiple network monitoring needs, emphasizing the practicality and efficiency of your implementation.
5.2 Task Outline 
5.2.1 Task I: Count-Min Sketch [30 points] 
• Implement the CountMinSketch class in task1 .py.
• This task focuses on implementing the basic Count-Min Sketch algorithm, providing a foundational understanding of how sketches work.
• The learning outcomes include understanding the update and inference processes of a Count-Min Sketch and recognizing its limitations, particularly in terms of accuracy across different applications.
5.2.2 Task II: Elastic Sketch [40 points] 
• Implement the ElasticSketch class in task2 .py.
• This task requires you to implement the sketch according to the design principles outlined in the ElasticSketch paper [5].
• The goal is to achieve higher accuracy across various network monitoring tasks, ad- dressing the limitations encountered in Task I.


5.2.3 Task III: Elastic Sketch Extended [30 points] 
• Implement the ElasticSketchExtended class in task3 .py.
• This task extends the ElasticSketch to achieve higher granularity in heavy hitter de- tection, while maintaining the capabilities from Task II.
• In this scenario, consider a network under a DDoS attack. Alongside the functionality achieved in Task II, the sketch should have the additional capability to query for heavy hitters based on Source IP, Destination IP, Source Port, and Destination Port.Each task is designed to be a stepping stone for the next, allowing you to reuse and refine your code as you progress.  The focus is on achieving accurate estimations and optimizing memory usage within the constraints provided.  A common class structure is reused across all tasks to ensure consistency and modularity where possible in your implementation.
5.3 Initializing the Sketch 
In addition to the functions described above, the following class methods must be completed:
• __init__ (self): This method is used to initialize the sketch according to your design. Use the self .sketch attribute to store your sketch data structure.  Additionally, you may use the self .auxiliary_storage  attribute for any auxiliary data structures you deem necessary. The combined memory for both the sketch and auxiliary storage must not exceed 1 Megabyte. You can define your own initialization parameters and adjust the class object initialization accordingly.
• hash_func(self,  packet_flow,  i=None): Implement this function to return the hash val- ues corresponding to a given flow.   The  optional argument  i  allows  you to specify the hash function ID, which is particularly useful when multiple hash functions are required.
• add_item(self,  packet_flow,  packet_len): This function is used to update the sketch as packets flow through the network. The function accepts packet_flow, representing the 5-tuple header fields, and packet_len, indicating the packet size, to update the sketch accordingly.
Important Notes:
1.  Please only modify the parts of the code that are specifically marked to be com- pleted in the skeleton code provided.  Do not modify the skeleton code structure itself.
2. Do not use/initialise any additional data structures in any of the functions except for the __init__ () method.
3. If you initialize any additional instance attributes (e.g., self.extra_var) in the __init__ () method, please ensure that the memory usage of these attributes is included in the assertion statement at the end of the method to ensure your overall memory usage does not exceed the given budget.




6    Testing Your Implementation

To facilitate basic testing of your implementation, we provide some preliminary test cases. Please note that these test cases are intended for self-evaluation only. Passing all provided test cases does not guarantee maximum grade. Your code will also be evaluated against hidden test cases and unseen network traces.
You can test the functionality of your code by running the following command:
python test.py  
Here, file_name corresponds to the trace file you are testing against, located in the Data/folder. The sketch_type parameter refers to the sketch being queried and accepts the following values: 'CountMin', 'ElasticSketch', and 'ElasticSketchExtended' for Task I (Section 5.2.1), Task II (Section 5.2.2), and Task III (Section 5.2.3), respectively.
For example, to test the trace in Data/packet_trace_main.csv for Task II, you would run: python test.py packet_trace_main.csv ElasticSketch.








         
加QQ：99515681  WX：codinghelp
