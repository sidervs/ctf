# Write-up

This was a stripped golang binary, which is a mess.
When running the binary and entering some input, we get "Nope."
So, we can search for the string to get the main function.

Searching a sequence of bytes for "Nope." reveals:



Now, we can see where the bad-boy is. Close to it is the good-boy message.


The flag is checked for length 42, and if it is - it is then checked byte by byte.
I didn't bother with the check as this is only a few thousands checks. Instead, I used gdb to break on the location of the bad-boy, checked the current index of the fail, and changed the flag accordingly.

gdb_script : 
```
b * 0x047BA23
commands
    printf "counter = %x\n\n", *(int *)($rsp+0x38)
    q
end
r
```
sol.py:
```
import subprocess

FILENAME = "./main_strip"
SPACE = ' '
COUNTER_STR ='counter = '
BAD_BOY = 'Nope.\n'
flag = list("hxp{"+SPACE*(42-5) +"}")

count = 4
count_old = 4
# Find all but 1 char
while count < len(flag)-2:
    r = subprocess.Popen(['gdb','-x','gdb_script','--args',FILENAME, ''.join(flag)], stdout=subprocess.PIPE)
    output = r.communicate()
    counter_loc = output[0].find(COUNTER_STR)+len(COUNTER_STR)
    counter_end = output[0].find('\n', counter_loc+1)
    count = int(output[0][counter_loc:counter_end],16)
    if count != count_old:
        print "Got {0}, current flag ={1}".format(count, ''.join(flag))
        count_old = count
    flag[count] = chr(ord(flag[count])+1)

# Find last unknown char
while(True):
    r = subprocess.Popen([FILENAME, ''.join(flag)], stdout=subprocess.PIPE)
    output = r.communicate()
    if BAD_BOY == output[0]:
        flag[count] = chr(ord(flag[count])+1)
    else:
        print output[0]
        print ''.join(flag)
        break
```
The flag was: hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5_S4F3}

output:
```
Got 5, current flag =hxp{k                                    }
Got 6, current flag =hxp{k3                                   }
Got 7, current flag =hxp{k3e                                  }
Got 8, current flag =hxp{k3eP                                 }
Got 9, current flag =hxp{k3eP_                                }
Got 10, current flag =hxp{k3eP_C                               }
Got 11, current flag =hxp{k3eP_C4                              }
Got 12, current flag =hxp{k3eP_C4l                             }
Got 13, current flag =hxp{k3eP_C4lM                            }
Got 14, current flag =hxp{k3eP_C4lM_                           }
Got 15, current flag =hxp{k3eP_C4lM_A                          }
Got 16, current flag =hxp{k3eP_C4lM_An                         }
Got 17, current flag =hxp{k3eP_C4lM_AnD                        }
Got 18, current flag =hxp{k3eP_C4lM_AnD_                       }
Got 19, current flag =hxp{k3eP_C4lM_AnD_D                      }
Got 20, current flag =hxp{k3eP_C4lM_AnD_D0                     }
Got 21, current flag =hxp{k3eP_C4lM_AnD_D0n                    }
Got 22, current flag =hxp{k3eP_C4lM_AnD_D0n'                   }
Got 23, current flag =hxp{k3eP_C4lM_AnD_D0n't                  }
Got 24, current flag =hxp{k3eP_C4lM_AnD_D0n't_                 }
Got 25, current flag =hxp{k3eP_C4lM_AnD_D0n't_P                }
Got 26, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4               }
Got 27, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n              }
Got 28, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1             }
Got 29, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c            }
Got 30, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c_           }
Got 31, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__          }
Got 32, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G         }
Got 33, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0        }
Got 34, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_       }
Got 35, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i      }
Got 36, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5     }
Got 37, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5_    }
Got 38, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5_S   }
Got 39, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5_S4  }
Got 40, current flag =hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5_S4F }
Seems like you got a flag...

hxp{k3eP_C4lM_AnD_D0n't_P4n1c__G0_i5_S4F3}

```
