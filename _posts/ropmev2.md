#### Summary

This is a recently retired pwn challenge from hackthebox. I will be using Cutter for reverse engineering this binary

#### Binary's protection

Open the binary using Cutter

![source-01](/img/Screenshotr5.png){: .align-left}

Select the aaa option to analyze the binary 

![source-01](/img/Screenshotr6.png){: .align-left}

aaa command  execute other below commands to analyze the binary.

    + aa - alias for af@@ sym.*;af@entry0;afva
    + aac - analyze function calls (af @@ `pi len~call[1]`)
    + aar - analyze len bytes of instructions for references
    + aan - autoname functions that either start with fcn.* or sym.func.*
    + afta - do type matching analysis for all functions


+ Binary has a non executable stack

