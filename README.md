﻿# ProjectFANTOM

\documentclass[twocolumn]{article}
\usepackage{graphicx} % Required for inserting images
\usepackage{algorithm} 
\usepackage{amsmath}
\usepackage{algpseudocode} 
\usepackage{fancyhdr}
\renewcommand{\headrulewidth}{0pt}
\renewcommand{\footrulewidth}{0pt}
\usepackage[square,numbers]{natbib}
\bibliographystyle{abbrvnat}

\begin{document}
\begin{titlepage}
   \begin{center}
       \vspace*{1cm}

       \textbf{\huge Project FANTOM}

       \vspace{0.5cm}
        Flying, Autonomous, Neural Network, Training and Operational Machine
            
       \vspace{1.5cm}

       \textbf{\large Anna Lucille Breck}
       
       \vspace{.25 cm}
       

       \vfill
            
       A Neural Network approach to autonomous flying for the F-15E Strike Eagle
            
       \vspace{0.4cm}


    \centering
    \includegraphics[width=4cm]{f15.jpg}
  
     

            
       CSCI 4511W\\
       University of Minnesota\\
       Dan Challou, Ph.D.\\
       December 2023
            
   \end{center}
\end{titlepage}














\begin{abstract}


    
The experiment outlined in this document explores the application of Neural Networks to autonomously pilot an F-15E Strike Eagle aircraft through a preset mission within the DCS World flight simulator. The primary objective is to reduce the risk to human lives during missions by eliminating the need for a pilot and Weapons Systems Officer (WSO). The experiment involves a comprehensive approach, utilizing five Neural Networks to control various aspects of the aircraft, such as pitch, bank, altitude correction, thrust, and target engagement. The training process incorporates a Genetic Algorithm, and the performance evaluation involves analyzing errors using Markov's Process and least squares methods. The experiment yields promising results, demonstrating the potential for Neural Networks to effectively navigate and control complex aircraft in simulation environments. The insights gained from this experiment contribute to the ongoing exploration of autonomous systems in aviation.
\end{abstract}

\section{Introduction}



The McDonnell Douglas F-15E Strike Eagle was first introduced in 1988 as a long-range, high-speed bomber that comes equipped with electronic warfare systems \cite{f15}. It is capable of jamming signals, eliminating enemy fighters, and dropping payloads without the need for an escort. Due to its incredible capabilities, the F-15E became a crucial component of the Air Force's weaponry. However, the F-15E requires two airmen to operate: a pilot to maneuver the aircraft and a Weapons Systems Officer (WSO) to navigate, target, and drop the payload. This puts more lives at risk during missions, if the F-15E were to be shot down, two lives would potentially be lost instead of one. To address this issue, an experiment is being conducted that aims to use a Neural Net to autonomously pilot an F-15E through a preset mission. This would eliminate the need for a pilot and WSO, therefore saving manpower and lives.

Neural Networks are a type of machine learning algorithm that imitates the human brain by replicating neurons and how they form connections. Due to these properties, Neural Networks are the best solution to replace airmen, as they replicate how a person thinks, adapts, and reacts. The structure of a Neural Network consists of layers of neurons that take in data through activation functions. These neurons are connected to neurons of other layers through weight matrices. The process of a Neural Network is intuitive and easy to follow. Data is given to a Neural Network in either vector or matrix form. Each data point follows a weighted path to neurons through matrix multiplication. Each entry of the resulting vector is passed through an activation function, which defines the value of the neuron. These vectors of neurons are layers, also known as hidden layers, as they are hidden in the middle of the network and are used to shape and connect the data to produce an output. The basic structure of a Neural Network is pictured in Figure 1.

    \begin{figure}[htp]
    \centering
    \includegraphics[width=6cm]{NN.jpeg}
 \caption{Neural Net Structure}
    \label{fig:NN}
\end{figure}
     
\section{Related Work}

There have been several experiments implementing Neural Networks to self-driving mechanisms throughout the many years since their development in the 1960s. The experiment I am conducting draws from two in particular, Princton's DeepDriving, and a Neural Net experiment posted to the DCS forums. 


\subsection{DeepDriving}

A team of researchers from Princeton conducted an experiment where they created and trained a Convolutional Neural Network (ConvNet) to drive a racing simulator. They employed a behavior reflex approach to train the ConvNet, which involves directly mapping the sensory input to a series of affordance indicators to make driving decisions using a Caffe model. Caffe is a neural network structure made by Berkeley that has several models created by various research teams. For this particular experiment, the researchers used AlexNet, a well-developed image recognition neural net, to recognize cars and lanes.

To complete the experiment, a driver drove a test car through several different tracks in the racing simulator TORCS. During the drive, the team recorded the driving commands and affordance indicators, as well as several screenshots. These images were then passed through the AlexNet to map each image to affordance indicators. Driving commands like speed control and steering were then computed using the mapped affordance indicators.

\subsection{Neural Network Fun With DCS}

A different approach was shared on the DCS forum \cite{knuthwebsite} that involved multiple neural networks working together to control the aircraft based on data provided by the DCS simulator. This data included speed, position, waypoint information, and the current pitch, roll, and yaw of the aircraft. These pieces of information were referred to as "affordance indicators" for this particular approach. The neural networks were assigned random weight matrices and then subjected to a genetic algorithm to determine their fitness based on factors such as physics and time taken to reach the waypoints. This process is repeated until an algorithm is given that best meets the criteria. There is no scholarly documentation on this approach, only videos showing the neural network testing process and the results. 



\section{Approach}
To obtain the most precise representation of an F-15E, I opted to run a test using the advanced simulator software known as DCS World. This platform is widely used during the initial training process in the F-15E B-course for pilots and WSOs, making it the most accurate publicly available simulation. Due to its complexity and accuracy, DCS World has a select community of aircraft enthusiasts who possess extensive knowledge in all technical aspects of the aircraft. In light of this, the developers of DCS have incorporated numerous ways to customize missions and gather data, allowing real or replica aircraft control components to be used when flying the simulated aircraft. The Developers have also made customizing missions easy as well, allowing for action points, also known as waypoints to be set wherever the user desires. The location data of these waypoints (time to, position, action) can be returned through the integration system the DCS developers came up with.
\subsection{Lua Integration}
The developers created a file named "Export.lua" with ease of integration in mind. This file contains functions that enable transferring and inserting data via sockets. By connecting your controllers and instruments to the simulator, you can enjoy a more realistic experience. Not only does this enable data collection to be sent to the Neural Net, but it also allows the Net to communicate back, which is beneficial for the experiment.

To start the process, the function \textit{LuaExportStart()} opens the connection to the port where the Neural Net will receive and send the data. The function \textit{LuaExportActivityNextEvent(t)} takes in a time variable $t$, collects data, and sets it through the socket. The function works by first comparing the input time (the current sim time) to a global time variable that can be set, if the current time matches the global variable then the function proceeds with collecting data. The collected data includes the plane's current position in the DCS coordinate system (x, y, z), altitude, angle (pitch, roll, yaw), current airspeed, and current waypoint position. The global time variable is adjusted based on the distance of the current waypoint. If the plane is near the point, the time variable is adjusted slightly (0.5 seconds), while if it is far away, the adjustment is larger (10 seconds).

The function \textit{LuaExportBeforeNextFrame()} checks for socket activity (commands sent from the Neural Network) and relays those commands to the simulator using the function \textit{LoSetCommand(command, value)}. The input parameter command reads from a list of commands controlling each part of the aircraft. However, for this experiment, we are only concerned with includes joystick control, thrust control, weapon firing, and payload drop.
\subsection{Neural Network Structure}
As the aircraft's flight requires a large amount of data and its complexity cannot be addressed by a single Neural Net, I decided to break down the problem into five Neural Nets. The first Net receives the vector distances, $y$, and $x$, from the aircraft's current position to the next waypoint to return the $\theta_p$ (pitch) and $\theta_y$ (yaw) values required to get the aircraft to that particular point. If the aircraft needs to climb, then the pilot must point the nose upwards, which results in a positive y-vector. Conversely, if the aircraft needs to descend, then the pilot must point the nose downwards, resulting in a negative y-vector. Since the F-15E does not have rudders like other aircraft, it relies solely on bank turns, rendering the yaw value from the first Neural Net useless. Therefore, the second Neural Net takes into account the yaw from the first, the current airspeed $s$, and the radial distance to the waypoint $r$, to calculate the bank angle necessary for the turn.  Turning left or right is achieved by tilting the wings in the corresponding direction. For instance, to turn right, the left-wing goes up, and to turn left, the right-wing goes up. This is determined by looking at the yaw value; if it is positive, the left-wing should go up, and if negative, the right-wing should go up. According to the F-15E flight manual \cite{f15man}, the aircraft's altitude must be kept constant during the turn, which is handled by the third Neural Net. This Neural Net considers the current pitch, altitude, and altitude difference between the aircraft and the waypoint, allowing for the pitch to be corrected based on the current values. To ensure that the aircraft flies smoothly, the fourth component deals with thrust. This component takes into account the current airspeed, pitch, and bank values, as well as the radial distance to the waypoint. It then calculates and returns the appropriate thrust value that should be set. 

The last neural network can be a bit tricky to handle, as it requires several inputs. These inputs include the current airspeed and position of the ego aircraft, as well as information on the locked target. If there is no locked target, this input will be labeled as "None." If there is a target, its position and velocity will be taken into account, with a velocity of 0 indicating a ground bombing target. Once all inputs are in place, the neural network will give either the fire command (if the velocity is not 0) or the drop command (if the velocity is 0). This ensures that the payload or firing will be done efficiently.


I used Neural Nets with two layers and 8 nodes per layer since the data input was not extensive. For the first three layers, I chose the hyperbolic tangent function to enable both negative and positive values. For the fourth layer, I used the sigmoid function, and for the fifth layer, I used the step function.
\subsection{Markov Process}
Markov Process is a discrete probabilistic process that determines future states based on the current configurations known as Markov Chains. These chains are constructed of the vector of probabilities \textbf{u}$^{(k)}$ demonstrating the $k^{th}$ state vector and the transition matrix $T$ whose entries show the transition probabilities. Together these values demonstrate the probability that the next state will occur in the form of \cite{linalg}
\begin{equation}
    \text{\textbf{u}}^{(k+1)} = T\text{\textbf{u}}^{(k)}
\end{equation}


To achieve optimal outcomes for the initial weight matrices $W_i$, I utilized an approach that treats each weight matrix as a transition matrix and layers as probability vectors. This approach ensures that the values start smaller and more polarized. The Markov Process requires the transition probabilities associated with each value of the vector to add up to 1 and remain positive. For the initial Neural Nets, I set each row to equal one and swapped the values to correspond to different tests. Essentially, every weight matrix will possess this property:
\begin{equation}
W_i = 
    \begin{bmatrix}
    w_{0,0}& \dots &w_{0,n}\\
    \vdots& \ddots& \vdots\\
    w_{n,0}& \dots &w_{n,n}\\ 
    \end{bmatrix},    w_{0,0} + \dots +w_{0,n} = 1
\end{equation}

I then can take these and swap them to ensure that every matrix is unique for the tests. 


\vspace{.4cm}

The output values of the Neural Nets are going to be trained to fit the commands for pitch (nose up/down), bank (wing position), and thrust, meaning that instead of going out as perfect angular values, they will come out as decimals between $-1$ and $1$, corresponding to the amount the joystick/ thrust lever must be moved. The target Net will either give the command or not give the command. 

\section{Experiment}

To allow the neural network to fly the DCS simulator, it was necessary to calibrate the weights accurately. The testing process involved creating 

multiple neural networks, having them fly in a unity simulator concurrently with preset waypoints, and then testing them using a genetic algorithm to identify the most successful Neural Nets. This procedure was repeated with newly generated Neural Nets until one was found that could complete a mission.

\subsection{The Unity Simulator}
I used Unity Physics, which is a built-in physics program in Unity, and a 3D model of an F-15E to develop a method for testing multiple Neural Nets simultaneously. To begin with, I inserted preset waypoints and generated several F-15E models to match each batch test. Initially, the first generated Nets did not contain many successful aircraft, with most of them crashing. However, as each test progressed, the results got better and better through the Genetic algorithm testing.


\subsection{Genetic Algorithms}

Genetic algorithms are algorithms designed around the idea of natural selection, a population eliminating certain genes based on survival. In computer coding this idea centers around optimization, first, a population is set up containing a set of individuals. These individuals have characteristics referred to as  genomes, which can be mutated and altered to form new members. In line with natural selection the population is put through a test referred to as a fitness function, this function takes in a genome, values for testing, and a target value. The idea of the fitness 
functions I used for the testing process are given in the pseudocode, Algorithm 1. The Neural Networks are tested on time taken, crashes, physics, and target hits, the Networks that pass are set to the next phase for the generation of the new population. 

During the Genetic algorithm process, the next phase is referred to as the crossover function. Its purpose is to splice together the top few individuals of the population to create the population for the next generation. In the experiment, this process is achieved by taking each weight matrix of the remaining population, identifying the minimum and maximum of each entry, and then creating new matrices 
based on that information. The \textbf{numpy.random} library is utilized specifically to carry out this task.

The final step in genetic algorithms is mutation. This process involves randomly altering the new genomes to ensure genetic diversity. To achieve this, I selected random values while integrating over the matrices, and transformed them into another decimal float value, also chosen at random. This helps to ensure that the new solutions can ensure new random solutions, just like in real like how some mutations like the lack of wisdom teeth can benefit the population as a whole. 

\begin{algorithm}
	\begin{algorithmic}[1]
		\For {Neural Network in Population}
			\State Compare the result to the target
			\If{result is bad}
				\State Remove Neural Network from population
                \EndIf
			\EndFor
        \State Remaining Networks in the Population are then sent to generate the new population
	\end{algorithmic} 
 \caption{Fitness Function} 
\end{algorithm}

\vspace{.4cm}
After multiple iterations, a Neural Network was developed that could complete a mission in DCS World.



\section{Analysis}

To assess the effectiveness of the Neural Net, the final step was to compare its performance to that of an actual pilot and calculate the error. This was achieved by using "Export.lua" to record the various position variables as well as the real pilot's actions and compare them to the actions taken by the neural net. The errors were computed in two ways, Markov's Process error and least squares error.

\subsection{Markov's Process error}

Markov's Process error is calculated based on the subdominant eigenvalue, $\lambda_2$. This value is determined by working backward through the weighted matrices from the pilot's value. The process for each matrix looks like this: 
\begin{equation}
    (W_i - \lambda I) = e
\end{equation}


where n eigenvalues end up being the diagonal values of the identity nxn matrix, I. The table below shows the average subdominant eigenvalue of each Neural Net, which serves as an indicator of Markov's error. (Each data point is an average of points taken and was analyzed by a simple script I wrote to backpropagate) 
\vspace{.4cm}

\begin{tabular}{c||c|c|c|c}
     $\lambda_2$ & Pitch & Bank & Altitude Corr. & Thrust \\
     \hline
     $W_3$ & .248 & .422 & .115 & .535\\
     $W_2$ & .135 & .332 & .135 & .443\\
     $W_1$ & .156 & .589 & .212 & .296\\
     \hline
     $\bar\lambda_2$ & .180 & .448 & .154 & .425 \\
\end{tabular}

\vspace{.4cm}

After examining the data, it appears that the accuracy of thrust and bank is lower when compared to an actual flight. This is expected because bank turns can be executed at any angle between $\frac{\pi}{6}$ and $\frac{\pi}{3}$ based on the pilot's preference and external factors like speed, wind, and G-force. The thrust value can also vary based on the pilot's preference, but the crucial aspect is to ensure that the cockpit is not subjected to more than 8Gs during the turn.

\subsection{Least Squares Error}

Another way to quantify error is with a least squares approximation because the Markov chain was only used as a baseline for the first, not for the weighted matrices that came from the genetic algorithm. This is why we see values as high as 40\% in the error. To compute the least squares approximation the formula below is used to compute the overall error in commands given to the simulator. 
\begin{equation}
    \sum |p_i - n_i|^2
\end{equation}
where $p_i$ is the value the pilot gave to the simulator and $n_i$ is the value given by the Neural Net. Again, each value was recorded and sent through as a script to be part of the average as there were several data points taken throughout the mission. 

\vspace{.4cm}

\begin{tabular}{c|c|c|c||c}
     Pitch & Bank & Altitude Corr. & Thrust & Total\\
     \hline
     .012 & .063 & .008 & .070 & .153
\end{tabular}

\vspace{.4cm}

Calculating error this way gives a total error closer to the actual value as no back-propagating needs to be done and the final matrices are not Markov chains.

\section{Conclusion}
I am happy with the outcome of the experiment as the least squares error was only about 15\%. However, I believe that the experiment can be improved by testing it on a computer with more GPU processing power. My computer would freeze up when running the Neural Net and the simulator simultaneously, which made it so I couldn't actually test the Neural net on the sim, only against data given to me by pilots. I am happy with the Genetic Algorithm approach to testing, as it showed mein a more intuitive way how Neural Nets narrow down the value of their connections between layers. 


\onecolumn
\nocite{*}
\bibliography{references}


\end{document}
