# Custom-Linux-Shell
/* Copyright [2020] <Khawaja Ahmad>
 * 
 */

/* 
 * File:   main.cpp
 * Author: Khawaja Ahmad
 *
 * Created on February 21, 2020, 5:17 PM
 */

#include <cstdlib>
#include <string>
#include <iomanip>
#include <iostream>
#include <vector>
#include "unistd.h"
#include "boost/algorithm/string.hpp"
#include "sys/wait.h"
#include "fstream"
#include "boost/algorithm/string/trim.hpp"
using namespace std;
/**
 * This method replaces running program with a different program.
 * But process does not change.
 * @param argList takes in a string vector containing a list of arguments.
 * @return on success exec calls never return 
 */
void myExec(vector<string> argList) {
    vector<char*> args;
    for (auto& s : argList) {
        args.push_back(&s[0]);  // address of first character
    }
    args.push_back(nullptr);
    execvp(args[0], &args[0]);    
}

/**
 * This method forks and executes, by calling the myExec method.
 * @param strVec takes in a string vector containing an argument list
 * @return the return type is void. 
 */
void forkNexec(const vector<string> strVec) {
    // fork and save the pid of the child process 
    int childPid = fork();
    // call the myExec helper method in the child 
    if (childPid == 0) {
        myExec(strVec);
    } else {
      // parent waits for the child process to finish
      // call on the wait method and get the exit code  
      // int exitCode = wait(childPid);
      int exitCode = 0;   
      waitpid(childPid, &exitCode, 0);
      cout << "Exit code: " << exitCode << endl; 
    }
}

/**
 * This is a helper method which helps both serial and parallel methods
 * deal with extracting data from the string stream and storing it in a vector 
 * @param ss takes in a reference to the string stream 
 * @return this method returns string vector containing data extracted from 
 * the string stream 
 */
vector<string> lineProcessing(istringstream& ss) {
     // a vector to store each line 
    vector<string> list;
    // a string word used to push stream into the vector
    string word;
    while (ss >> quoted(word)) {
                list.push_back(word);
            }    
    return list;
}

/**
 * This method runs the serial commands in a file.
 * The shell waits for each command to finish. 
 * @param this method takes in a constant 
 * reference to the file name 
 * @return the return type is void 
 */
void serial(const string& fileName) {
     std::ifstream input(fileName);
    string line; 
    while (getline(input, line)) {
        // if it's not a comment and the line is not empty 
        if (!line.empty() && line.at(0) != '#') {
            // convert the line into a string stream 
            istringstream myLine(line);
            // send the string stream to the helper method to take care 
            // of line processing
            vector<string> list = lineProcessing(myLine); 
            cout << "Running: " << list[0] << " " << list[1] << endl;   
          forkNexec(list);         
        }
    }
}
/**
 * This is a helper method written to aid with parallel command execution.
 * It waits for all processes to finish in the same order they were listed. 
 * @param pids this method takes in an int vector containing process ids.
 * @return the return type is void
 */
void wait(vector<int> pids) {
    for (auto i : pids) {
      int exitCode = 0;   
      waitpid(i, &exitCode, 0);
      cout << "Exit code: " << exitCode << endl; 
    }
}
/**
 * This method runs the parallel commands in a file.
 * All the commands are first executed and then it 
 * waits for all the processes to finish, in the same order they were listed
 * @param this method takes in a constant 
 * reference to the file name 
 * @return the return type is void 
 */
void parallel(const string& fileName) {
    std::ifstream input(fileName);
    string line; 
    vector<int> pids;
    while (getline(input, line)) {
        // if it's not a comment and the line is not empty 
        if (!line.empty() && line.at(0) != '#') {
            // convert the line into a string stream 
            istringstream myLine(line);
            // send the string stream to the helper method to take care 
            // of line processing
            vector<string> list = lineProcessing(myLine); 
            
    cout << "Running: " << list[0] << " " << list[1] << endl;  
    int childPid = fork();
    // if it's the child, execute it 
    if (childPid == 0) {
        myExec(list);
    // if it's the parent, store the pid     
    } else {
      // store the pid in a vector of pids
      pids.push_back(childPid);
           }  
    }
}  
    wait(pids); 
}

/**
 * The main method takes in input from the user and depending on the command,
 * it calls the appropriate methods. if the command is exit, it terminates the shell.
 * if the command is serial, it calls the serial method.
 * if the command is parallel, it calls the parallel method.
 * if it's not one of the above three, then it is assumed to be the 
 * name of the program to be executed. 
 * @param argc the number of command line arguments
 * @param argv the actual command line arguments 
 */
int main(int argc, char** argv) {
    string line;
    while (cout << "> ", getline(cin, line)) { 
        if (!line.empty() && line.at(0) != '#') {
            // remove quotation marks
            line.erase(remove(line.begin(), line.end(), '\"'), line.end());
            // removing trailing and leading spaces
            boost::algorithm::trim(line);
            // a vector to store the arguments
            vector<string> list;
            // split the arguments on space and store them
            // in a vector
            boost::split(list, line, boost:: is_any_of(" "));
            // if it's the serial command
           if (list[0] == "exit") {
               break;
            // if it's the parallel command
           } else if (list[0] == "PARALLEL") {
               parallel(list[1]);
             // if it's the exit command 
           } else if (list[0] == "SERIAL") {
               serial(list[1]);
           } else {
               cout << "Running: " << line << endl;
                forkNexec(list);
           }
        }
    }
    return 0;
}


