#!/usr/bin/python
""" 
   Generate a flat version of LaTeX document with indexing commands inserted
   Author: Mehmet Gencer, mgencer@cs.bilgi.edu.tr

   The program does two things:
   (1) Takes a Latex input file, if it contains \input{} directives, the external files are inserted in place of these directives.
   (2) The index terms are parsed from a given text file which has a special format and wherever the subject terms are found in the original text, \index{} directives are inserted.

   The subject index format allows one to describe alternatives and sub indexing. Separate alternatives with "/", levels with ">" or "<" depending on word order, and optional labels within "[" and "]" if different from the index term. 
   e.g.::
   communities
   communities>of practice
   communities<professional
   occupational communities
   agent-based/agent based
   [network/networks]network/networks>of developers

   When a label is not specified within [ and ], the first alternative will be used. i.e. "agent-based" will be the index label, versus label "network/networks".

   The output is the terms succeeded with indexing commands such as:
     \index{hello} 
     \index{hello!Peter}

   texindex excludes verbatim environments from being processed for indexing.
"""
import sys, re, string, os.path, getopt

inputdir=""
lines=[] #lines of text
CUSTOM=0 #make custom fixes for my thesis
def debug(*args):
    msg=""
    for a in args:
        if msg:msg+=" "
        msg+=str(a)
    sys.stderr.write(msg)
    sys.stderr.write("\n")
def debug2(*args):
    pass
def insertfile(fname,custom=0):
    for rline in open("%s/%s"%(inputdir,fname),"r").readlines():
        line=rline.rstrip()
        if custom:
            if line.find("%%\\")==0:
                line=line[2:]
        if custom>=2:
            for i in range(2,custom+1):
                if line.find("%"+"C"*i+"%\\")==0:
                    line=line[i+2:]
                    break
        if line.find("\\input{")==0:
            insertfilename=line.split("{")[1].split("}")[0]
            insertfile(insertfilename)
        else:
            lines.append(line)

class Term:
    def __init__(self,id,initlineraw):
        self.id=id
        self.markstart=self.mark=":::TERM%dsss"%self.id
        self.markend=":::TERM%deee"%self.id
        self.indexterm=initlineraw.split(":")[0]
        try:
            alts=initlineraw.split(":")[1].strip()
        except:
            alts=""
        if not alts:
            alts=self.indexterm.replace("!"," ")
        self.alternatives=alts.split(",")
        debug(self.indexterm, self.alternatives)


    def render1(self,text):
        retval=text
        pat=""
        for a in self.alternatives:
            if pat:pat+="|"
            pat+=a
        x="(^[^\\\\].*\s+)(%s)([\s\.,:;])"%pat
        debug("X:"+x)
        #raw_input()
        #c=re.compile(x, re.IGNORECASE)#|re.MULTILINE)
        #tmp=c.sub("\\1\\2%s\\3"%self.mark,retval)
        c=re.compile("(^[^\\\\].*[^\:\{\[\.]) (%s)([s\s\.,?:;])"%pat, re.IGNORECASE|re.MULTILINE)
        tmp=c.sub("\\1 \\2%s\\3"%self.mark,retval)
        retval=tmp
        return retval
    def render2(self,text):
        retval=text
        c=re.compile(self.mark)
        tmp=c.sub("\\\\index{%s}"%self.indexterm,retval)
        retval=tmp
        return retval
def markOutVerbatim(text):
    """
    Remove the verbatim environments, leaving marks instead, to protect 
    from indexing.
    Returns (newtext,verbatimrestore) where newtext is the text with verbatims removed
    and verbatim restore is a dictionary which can later be used to restore verbatim environments.
    """
    retags=re.compile("(\{\S+?\})")
    retval=""
    counter=0
    vd={}
    inverbatim=0
    vs=""
    for l in text.split("\n"):
        if l.find("\\begin{verbatim}")==0:
            inverbatim=1
            counter+=1
            vs=l+"\n"
        elif l.find("\\end{verbatim}")==0:
            inverbatim=0
            vs+=l+"\n"
            retval+="#TEXINDEXVERBATIM:%d#\n"%counter
            vd[counter]=vs
        else:
            if inverbatim:
                vs+=l+"\n"
            else:
                f=retags.findall(l)
                for x in f:
                    counter+=1
                    l=l.replace(x,"#TEXINDEXVERBATIM:%d#"%counter,1)
                    vd[counter]=x
                retval+=l+"\n"
    return (retval,vd)
def markInVerbatim(text,vd):
    retval=""
    retags=re.compile("#TEXINDEXVERBATIM:\\d+#")
    for l in text.split("\n"):
        f=retags.findall(l)
        for x in f:
            c=int(x.split(":")[1][:-1])
            l=l.replace(x,vd[c],1)
        retval+=l+"\n"
    return retval
if __name__=="__main__":
    global CUSTOM
    optlist,args=getopt.getopt(sys.argv[1:],"c:")
    try:
        infile=args[0]
        indextermsfile=args[1]
    except:
        debug("Usage:\n %s [-c] inputfile indextermsfile"%sys.argv[0])
        debug("Options:\n  -c : Custom formatting. Uncomment lines starting with %%\\")
        debug(__doc__)
        sys.exit()
    for o in optlist:
        o,v=o
        sys.stderr.write("%s,%s\n"%( o,v))
        if o=="-c":
            CUSTOM=1
            try:
                CUSTOM=int(v)
            except:
                pass
            sys.stderr.write("CUSTOM PROCESSING ENABLED: %d (%s)\n"%(CUSTOM,v))
            #raw_input()
    absinfile=os.path.abspath(infile)
    inputdir=os.path.dirname(absinfile)
    infilebase=os.path.basename(absinfile)
    insertfile(infilebase,custom=CUSTOM)
    text=""
    for l in lines:
        if text:text+="\r\n"
        text+=l
    terms=[]
    termcount=0
    for term in open(indextermsfile,"r").readlines():
        t=term.strip()
        if not t:continue
        if t[0]=="#":continue
        termcount+=1
        terms.append(Term(termcount,t))

    out,vd=markOutVerbatim(text)#out=text
    for t in terms:
        out=t.render1(out)
    for t in terms:
        out=t.render2(out)
    out=markInVerbatim(out,vd)
    print out
    try:
        sys.stderr.write(vd[-1])
    except:
        sys.stderr.write("No verbatim\n")
