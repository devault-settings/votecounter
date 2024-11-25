# This code is adapted from a votecounting code developed for use in elections 
# at Ricketts Hovse, one of the 8 undergraduate houses at Caltech.
# It exists thanks to the contributions of Coby Abrahams, Alejandro López, 
# Alison Dugas, Umesh Padia, Jack Briones, Audrey DeVault, and other Skurves.

from glob import glob
import copy
import os
import networkx as nx
import matplotlib.pyplot as plt
from collections import Counter
from warnings import simplefilter


# Loading the ballots, hard coded for the .csv format that google sheets outputs to.
# Note that since google exports to .csv DO NOT USE COMMAS in candidate titles
# Returns a list of lists, where the first entry defines which candidate
# that position in the ballots refer to, and the rest of the entries are the
# ballots themselves.
def LoadBallots(filename, remove_parentheses):
    spreadsheet = open(filename,'r')
    ballots = []
    ballot = ' '
    header = True
    while not ballot == []:
        ballot = spreadsheet.readline()
        ballot = ballot.split(',')[1:]
        if len(ballot) > 0:
            if len(ballot[-1]) > 2:
                ballot[-1].rstrip() # trimming end of line stuff
            if not header:
                # ballot = map(int,ballot) #commenting out this line just leaves the ballots as lists
                # which works fine as the Tennesee example shows
                if '\n' in ballot[-1]: # however you must add these lines to remove the '\n' from the last string in each
                    ballot[-1] = ballot[-1].strip('\n') # ballot, otherwise it'll miscount for the last listed candidate
        ballots.append(ballot)
        header = False
    ballots = ballots[:-1]
    headers = [i.rstrip() for i in ballots[0]]
    names=[]
    for name in headers:
        # Trimming out the actual candidate titles from the header
        # since google forms outputs each header as "Vote[Name]"
        if '[' in name:
            name= name.split('[')[1]
            name=name[:-1]
        if ']' in name:
            name= name.split(']')[0]
        # Also trimming out anything in parentheses after the name for use
        # in acronymn voting, can be commented out if parentheses are needed
        if remove_parentheses:
            if '(' in name:
                name= name.split('(')[0]
        names.append(name)
    ballots[0] = names
    return ballots

# Removes candidate i from all ballots in order to find a second place winner
def RemoveCandidate(ballots,i):
    for j in range(len(ballots)):
        ballots[j].pop(i)
    return ballots

# Makes example ballots accoring to the Tennessee example in the Ranked
# Pairs wikipedia page, assuming 100 votes
def MakeTennesseeBallots():
    memphis = [1,2,3,4]
    nashville = [4,1,2,3]
    chattanooga= [4,3,1,2]
    knoxville = [4,3,2,1]
    ballots = [['mem','nash','chat','knox']]
    for i in range(42):
        ballots.append(memphis)
    for i in range(26):
        ballots.append(nashville)
    for i in range(15):
        ballots.append(chattanooga)
    for i in range(17):
        ballots.append(knoxville)
    return ballots

# Returns a matrix where entry (i,j) is the number of ballots on which
# candidate i beats candidate j. If (i,j) = (j,i), both are set to zero
# to avoid problems later.
def CountBallots(ballots):
    scores = copy.copy(ballots[1])
    for i in range(len(scores)):
        scores[i] = copy.copy(ballots[1])
        for j in range(len(scores[i])):
            scores[i][j] = 0
    for ballot in ballots[1:]:
        for i in range(len(ballot)):
            for j in range(len(ballot)):
                if ballot[i] < ballot[j]:
                    scores[i][j] += 1
                elif ballot[i] == ballot[j]:
                    scores[i][j] += 0.5
    for i in range(len(scores)):
        for j in range(len(scores[i])):
            if scores[i][j] == scores[j][i]:
                scores[i][j] = 0
                scores[j][i] = 0
    return scores


# Identifies the largest value(s) in the scores matrix, adds that to a directed
# graph of candidates, where a->b means a beats b, and repeats. The graph is
# stored in a dictionary where (key,value) is key->value. An edge is not
# drawn if it would complete a cycle. The source of the graph, if there is a
# single one, is the winner. If there is a cycle where everyone in the cycle
# beats everyone outside the cycle, a runoff is needed between the members of
# the cycle. Note that a runoff will only work if people change their votes.
def ConstructGraph(scores):
    graph = {}
    maxindex = FindMax(scores)
    while(maxindex != [(-1,-1)]):
        if len(maxindex) == 1:
            if maxindex[0][0] in graph.keys():
                graph[maxindex[0][0]].append(maxindex[0][1])
            else:
                graph[maxindex[0][0]] = [maxindex[0][1]]
            if CreatesCycle(graph,maxindex[0][0],maxindex[0][0],0) == True:
                graph[maxindex[0][0]].pop(-1)
            scores[maxindex[0][0]][maxindex[0][1]] = 0
            scores[maxindex[0][1]][maxindex[0][0]] = 0
        else:
            for index in maxindex:
                if index[0] in graph.keys():
                    graph[index[0]].append(index[1])
                else:
                    graph[index[0]] = [index[1]]
                scores[index[0]][index[1]] = 0
                scores[index[1]][index[0]] = 0
                ifyourereadingthis = 0
                illbuyyouabeer = 0 # if 21+
        maxindex = FindMax(scores)
    return graph



# Returns tuple of indices (i,j) for the entry in the score matrix with the
# max value. If there is a tie, the tie is broken by which of the complementary
# locations, (j,i), indicating losing, is smallest. If there is still a tie,
# both are returned, and a runoff election becomes a possibility.
def FindMax(scores):
    maxscore = 0.5 # this way if every entry in scores is 0, it doesn't update
    maxindex = [(-1,-1)]
    for r in range(len(scores)):
        for c in range(len(scores[r])):
            if scores[r][c] > maxscore:
                maxscore = scores[r][c]
                maxindex = [(r,c)]
            elif scores[r][c] == maxscore:
                if scores[c][r] < scores[maxindex[0][1]][maxindex[0][0]]:
                    maxindex = [(r,c)]
                elif scores[c][r] == scores[maxindex[0][1]][maxindex[0][0]]:
                    maxindex.append((r,c))
    if len(maxindex) == 2 and maxindex[0][0] == maxindex[1][0]:
        if scores[maxindex[0][1]][maxindex[1][1]] > scores[maxindex[1][1]][maxindex[0][1]]:
            maxindex = [maxindex[0]]
        elif scores[maxindex[0][1]][maxindex[1][1]] < scores[maxindex[1][1]][maxindex[0][1]]:
            maxindex = [maxindex[1]]
    return maxindex

# Checks if the provided graph contains a cycle which contains the relevant
# new edge.
def CreatesCycle(graph, currentkey, origin,depth):
    if depth > len(graph.keys())+1:
        return False
    if currentkey in graph.keys():
        for key in graph[currentkey]:
            if key == origin:
                return True
            if CreatesCycle(graph,key,origin,depth+1):
                return True
    return False

# Draws the graph.
def DrawGraph(graph,names):
    fig,ax = plt.subplots(nrows=1, ncols=1)
    g = nx.DiGraph(graph)
    labels = {}
    for i in graph.keys():
        labels[i] = names[i]
    #Nodes which lose in all cases are not assigned a graph key,
    #so any remaining losing nodes are labelled here
    num_nodes=g.number_of_nodes()
    missing_nodes=[]
    for i in range(0,num_nodes):
        if i not in graph.keys():
            missing_nodes.append(i)
    for element in names:
        if element not in labels.values():
            labels[missing_nodes.pop(0)] = element
            
    nx.draw_networkx(g, pos=nx.circular_layout(g), alpha=0.9,  node_color='red', 
                     node_size=400, node_shape='*', width=1.5, edge_color='gray',
                     labels=labels, font_color='white', font_weight='heavy', arrowstyle='-|>',
                     arrowsize=15) #Lots of hard coded cutesy color/graph preferences here
    fig.set_facecolor("black")
    ax.axis('off')
    plt.show()

def RunTest(i=-1):
    ballots = MakeTennesseeBallots()
    if i != -1:
        ballots = RemoveCandidate(ballots,i)
    scores = CountBallots(ballots)
    graph = ConstructGraph(scores)
    names =  [i.rstrip() for i in ballots[0]]

    graph_keys = list(graph.keys())
    max_index = max(graph_keys)
    c = Counter(graph.keys())

    most_common = c.most_common(2)
    # print(most_common)

    print("the winner is " + names[max(graph.keys(), key=lambda k: len(graph[k]))])
    DrawGraph(graph, names)

def Run(filename,i=-1, remove_parentheses=False):
    if '.csv' not in filename:
        filename = glob('*{0}*.csv'.format(filename))[0]
        print(filename)
    ballots = LoadBallots(filename, remove_parentheses)
    if i != -1:
        ballots = RemoveCandidate(ballots,i)
    scores = CountBallots(ballots)
    graph = ConstructGraph(scores)
    names =  [i.rstrip() for i in ballots[0]]
    
# Uncomment if you want to print specifics on 1 v. 1 races
    # print(list(zip(names, CountBallots(ballots))))

    graph_keys = list(graph.keys())
    max_index = max(graph_keys)
    c = Counter(graph.keys())

    most_common = c.most_common(2)
    # print(most_common)

    print("the winner is " + names[max(graph.keys(), key=lambda k: len(graph[k]))])
    DrawGraph(graph, names)
