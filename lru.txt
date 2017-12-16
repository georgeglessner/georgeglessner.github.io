#!/usr/bin/env python

"""
A simple LRU simulation with specifications:
    - System physical memory is 16 KB
    - Page / frame size is 1 KB
"""

__author__ = "George Glessner"

import time
import random

input = ""  # input string from input file
frameList = []  # current frame list
processPageList = []    # list of all processes
pageTable = []  # keeps track of process' current page table
pfList = []  # list to keep track of page faults for processes
freeFrames = range(0, 16)    # free frame list
pCount = 0  # process count


def start():
    """ format input list and add to processPageList """

    global input, frameList, processPageList

    print "\n******************************************************************"
    print "\tThis is a simulation for LRU replacement."
    print "\tAt any time you may choose to quit('q') this program."
    print "******************************************************************"

    # Open File
    with open("input.txt") as f:
        original = f.readlines()

        # Remove ":", "\n"
        for line in original:
            input = line.replace(":", "")
            input = input.replace('\n', "")

            # Split by tab, add to processList
            output = input.partition('\t')

            # convert process to number
            pid = output[0]
            proc = int(pid[1])

            # convert binary page # to decimal
            page = int(str(output[2]), 2)

            # add process and page # to list
            processPageList.append([proc, page])


def run():
    """ run through input list, place processes in frame utilizing
        free frame list and LRU
    """

    global frameList, pageTable, processPageList, freeFrames, pCount
    inFrame, timeStamp, run = 0, 0, 0

    # run through each process in list
    for process in processPageList:

        # flag for reloop
        restart = True

        # timestamp increment
        timeStamp += 1

        # variable to denote whether in frame list or not
        inFrame = 0

        # variable to assign random frame
        randomFrame = random.randint(0, 15)

        # if list is empty (base case)
        if not frameList:
            pageFault(process[0])
            frameList.append([process[0], process[1], randomFrame, timeStamp])
            pageTable.append([process[0], process[1], randomFrame])
            for frame in freeFrames:
                if frame == randomFrame:
                    freeFrames.remove(frame)

        # run through each entry in frame list
        for entries in frameList:
            # if process id and page # are in frame list, set inFrame to 1
            if process[0] == entries[0] and process[1] == entries[1]:
                timeStamp += 1
                entries[3] = timeStamp
                inFrame = 1
                break

        # if not in frame
        if inFrame == 0:
            if freeFrames:
                while restart:
                    randomFrame = random.randint(0, 15)
                    # check through list of free frames
                    for frame in freeFrames:
                        # if frame is available, remove from free frames, append to frame list
                        if frame == randomFrame:
                            freeFrames.remove(randomFrame)
                            timeStamp += 1
                            pageFault(process[0])
                            frameList.append(
                                [process[0], process[1], randomFrame, timeStamp])
                            pageTable.append(
                                [process[0], process[1], randomFrame])
                            restart = False
                            break
                inFrame = 1

            # if frame list full, find least recently used
            elif not freeFrames:

                # initial values for LRU, index count, and frame to replace
                lru = 99999999
                count = -1
                frame = 0

                # set LRU for values in framelist
                for entries in frameList:
                    if entries[3] < lru:
                        lru = entries[3]

                # increase index, set frame for LRU
                for entries in frameList:
                    count += 1
                    if entries[3] == lru:
                        frame = entries[2]
                        break

                # remove least recently used from frame list, replace with new process
                timeStamp += 1
                frameList.remove(frameList[count])
                pageTable.remove(pageTable[count])
                pageFault(process[0])
                frameList.insert(
                    count, [process[0], process[1], frame, timeStamp])
                pageTable.insert(count, [process[0], process[1], count])

        # accept user input to step through or run to completion
        if run != 1:
            rinput = raw_input("\nEnter 'S' to step or 'R' to run: ")
            if rinput == 's':
                pCount += 1
                printCurrentStatus(process)
            elif rinput == 'r':
                pCount = len(processPageList)
                run = 1
            elif rinput == 'q':
                exit(1)
            else:
                print "\n**********************************************"
                print "Invalid entry, please enter 's', 'r', or 'q'"
                print "**********************************************"


def pageFault(proc):
    """ add page fault to list """

    global pfList
    pfList.append([proc, 1])


def printCurrentStatus(process):
    """ print current status """

    print "\n-----------------------------"
    print "Process / Page referenced"
    print "========================="
    print "Process: " + str(process[0])
    print "Page: " + str(process[1])

    print "\nPage Table"
    print "=========="
    print "Process " + str(process[0]) + ":"
    print "Page\tFrame"

    # get current process, print page+frame
    for x in pageTable:
        if x[0] == process[0]:
            print str(x[1]) + "\t" + str(x[2])

    print "\nPhysical Memory / Frame Table"
    print "============================="
    print "Frame# \tProcID\tPage# "

    # Check to see what process is in frame # and print
    for x in range(16):
        for frame in frameList:
            if frame[2] == x:
                print str(x) + "\t" + str(frame[0]) + "\t" + str(frame[1])
                break
        else:
            print str(x) + "\t\t"

    print "-----------------------------"


def printFinalStatus():
    """ print final status """

    finalProcess = processPageList[len(processPageList) - 1]

    # print output
    print "\n-----------------------------"
    print "Process / Page referenced"
    print "========================="
    print "Process: " + str(finalProcess[0])
    print "Page: " + str(finalProcess[1])

    print "\nPage Table"
    print "=========="
    print "Process " + str(finalProcess[0]) + ":"
    print "Page\tFrame"

    # get current process, print page+frame
    for x in pageTable:
        if x[0] == finalProcess[0]:
            print str(x[1]) + "\t" + str(x[2])

    print "\nPhysical Memory / Frame Table"
    print "============================="
    print "Frame# \tProcID\tPage# "

    # Check to see what process is in frame # and print
    for x in range(16):
        for frame in frameList:
            if frame[2] == x:
                print str(x) + "\t" + str(frame[0]) + "\t" + str(frame[1])
                break
        else:
            print str(x) + "\t\t"

    # if end of input file, print final page fault / reference statistics
    if pCount == len(processPageList):
        print "\nFinal Page Faults / References"
        print "=============================="

        # calculate page faults
        pflist = []
        for x in processPageList:
            count = 0
            for y in pfList:
                if x[0] == y[0]:
                    count += y[1]
            pflist.append([x[0], count])

        # sort page fault list
        pfset = set(map(tuple, pflist))
        pf = map(list, pfset)
        pf.sort(key=lambda x: pflist.index(x))

        # Calculate total references
        ref, finRef = [], []
        for x in processPageList:
            ref.append([x[0], 1])
        for x in processPageList:
            count = 0
            for y in ref:
                if x[0] == y[0]:
                    count += y[1]
            finRef.append([x[0], count])

        # sort reference list
        refset = set(map(tuple, finRef))
        ref = map(list, refset)
        ref.sort(key=lambda x: finRef.index(x))

        # print out total page faults / references
        for x in pf:
            for y in ref:
                if x[0] == y[0]:
                    print "Process", x[0], ':', x[1], "/", y[1]

        print "-----------------------------"


if __name__ == '__main__':
    start()
    run()
    printFinalStatus()
