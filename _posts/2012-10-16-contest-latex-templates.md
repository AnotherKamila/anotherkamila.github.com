---
title: My awesome contest LaTeX templates
layout: post
categories: [ latex ]
syntax_highlight: true
---

The time has come! Finally a new year with a lot of correspondence contests is here!
Here, a present for my contest-hungry peers: my LaTeX template, which I am so proud of.

<!--more-->

(Actually, there are several versions depending on what exactly it is; for now I am publishing the
KSP version.)

--------------------------------------------------------------------------------

preamble.tex:
{% highlight latex %}
\documentclass[10pt]{amsart}
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{amsthm}
\usepackage{graphicx}
\usepackage{helvet}
\usepackage{slovak}
\usepackage[utf8]{inputenc}
\usepackage{color}
%\usepackage{epic,eepic}
%\usepackage{cancel}
\usepackage{listings}
\usepackage{hyperref}
\usepackage[all]{hypcap}

\lstdefinelanguage{CoffeeScript}{
alsoletter={-=>?()[]\{\}},
keywords={if, then, unless, when, while, for, in, of, ->, =>, return, ?},
ndkeywords={class, new, [, ], (, ), \{, \}},
comment=[l]{\#},
morestring=[b]',
morestring=[b]",
morestring=[b]/
}

\lstset{
language=C++,
basicstyle=\scriptsize\ttfamily,
numbers=left,
numberstyle=\tiny\color{lightblue},
stepnumber=1,
numbersep=15pt,
showspaces=false,
showstringspaces=false,
tabsize=4,
breaklines=true,
keywordstyle=\color{darkblue}\bfseries,
ndkeywordstyle=\color{gray}\bfseries,
commentstyle=\color{gray}\em,
stringstyle=\color{red}
}

\hypersetup{
colorlinks,
citecolor=linkcolor,
filecolor=linkcolor,
linkcolor=linkcolor,
urlcolor=linkcolor
}

\definecolor{linkcolor}{rgb}{0, 0.4, 0.7}
\definecolor{blue}{rgb}{0,0.4,0.7}
\definecolor{darkblue}{rgb}{0,0.3,0.6}
\definecolor{gray}{rgb}{0.4,0.4,0.4}
\definecolor{red}{rgb}{1,0.05,0.05}
\definecolor{darkred}{rgb}{0.8,0,0}
\definecolor{lightblue}{rgb}{0.2,0.6,0.9}
\definecolor{black}{rgb}{0,0,0}

\renewcommand{\theenumi}{\arabic{enumi}}
\renewcommand{\labelenumi}{\theenumi.}

\setcounter{tocdepth}{1}

\newcommand{\hl}[1]{\textcolor{blue}{#1}}
\newcommand{\unit}[1]{\ensuremath \, \rm{#1}}

\newcommand{\fig}[4]{
\begin{figure}[h!]
    \centering%
    \includegraphics[width=#4\textwidth]{#1}
    \caption{#2}
    \label{fig.#3}
\end{figure}
}

\renewcommand{\qed}{\hfill \mbox{\raggedright $\Box$}}

\newcommand{\mytitle}[2]{
    \title{\hl{#1}}
    \author{
        Kamila Součková\\
        kategória #2\\
        ŠPMNDaG, Bratislava
    }
    \maketitle
}

\pagestyle{plain}
{% endhighlight %}

--------------------------------------------------------------------------------

solution_template.tex:
{% highlight latex %}
\input{./preamble.tex}

\begin{document}

\mytitle{}{O}

%\lstinputlisting{}

\end{document}
{% endhighlight %}

[Use, abuse, blame me!](/LICENSE)
