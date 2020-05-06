---
layout: post
published: false
title: clustal-netsurf
---
## Color-coding a clustal alignment file by NetsurfP2 values

[Netsurfp2](http://www.cbs.dtu.dk/services/NetSurfP/) produces many values for protein sequences including "q8" (classification of protein secondary structure into eight kinds) and "disorder" (a value indicating likelihood of a residue being in an intrinsically disordered region).

To display these values on top of a multiple sequence alignment is difficult, since the proteins may be different lengths and there will be gaps. This code reads a JSON file produced by NetsurfP2 and a CLUSTAL format file containing a subset of proteins contained in the JSON file, and produces a color-coded HTML file. 

Difficulties and limitations: take care that the names of the proteins in both files are the same; all periods in the names are converted to underscores and are truncated at 15 characters. If this collapses different protein names into the same name, the results will not be good.


import json
import argparse

def getjson(jsonfilename):
    jsonfile=open(jsonfilename)
    dat=json.load(jsonfile);jsonfile.close()
    q8={x['id'][5:20]:x['q8'] for x in dat}
    disorder={x['id'][5:20]:x['disorder'] for x in dat}
    return(q8,disorder)

def cluprint(clufilename,q8,disorder):
    darkness=50 #how dark the "disorder" color background of text should be relative to 255=total
    print('''
<!DOCTYPE html>
<html>
<head>

<style>
.G { color: red !important; }
.H { color: orange !important; }
.I { color: yellow !important; }
.T { color: green !important; }
.E { color: blue !important; }
.B { color: purple !important; }
.S { color: gray50 !important; }
.C { color: fuchsia !important; }
</style>

</head>

<body>

<h1>Clustal-Netsurf</h1>
<pre>
Color code (for q8):
Red: G (3-turn helix)
Orange: H (4-turn helix)
Yellow: I (5-turn helix)
Green: T (hydrogen-bonded turn)
Blue: E (extended strand in parallel and/or antiparallel beta-sheet)
Purple: B (residue in isolated beta-bridge)
Gray: S (bend)
Magenta: C (coil)

- Magenta background gradient correlates with higher disorder parameter

    ''')
    count={x:0 for x in q8.keys()}
    clufile=open(clufilename)
    clu=list(clufile);clufile.close()
    for line in clu:
        splitline=line.split()
        try:
            name=splitline[0].replace(".","_")
        except:
            name=""
        if name in q8.keys():
            spc=" "*(20-len(name))
            print(name,spc,end="")
            for char in splitline[1]:
                if(char != "-"):
                    bval="{:02x}".format(255-int(darkness*disorder[name][count[name]]))
                    bval="#FF"+(bval+"FF") #subtract from green depending on disorder
                    print('<span style="background-color: ',bval,'" class="',q8[name][count[name]],'">',char,'</span>',sep="",end="")
                    count[name]=count[name]+1
                else:
                    print(char,end="")
            print()
        else:
            print("    ",line)
    print('''
    </pre>
    </body>
    </html>
    ''')


def main():
    parser=argparse.ArgumentParser(description='Output a color-coded MSA file given clustal and json')
    parser.add_argument('clufilename',type=str, help='textfile containing a Clustal-format MSA')
    parser.add_argument('jsonfilename',type=str, help='textfile containing a JSON output of netsurfp2 of proteins in MSA')
    args=parser.parse_args()
    q8,disorder=getjson(args.jsonfilename)
    cluprint(args.clufilename,q8,disorder)


if __name__=="__main__":
    main()
