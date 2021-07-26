### A-Sokoban-Solver-Using-BFS
In this fun project I implemented Breadth First Search using Python to solve a "Push the Box" game that is quite popular on mobile. 
Remark: The notations for the layout of the game are borrowed from others (link:https://home.cse.ust.hk/~yqsong/teaching/comp3211/projects/2017Fall/G14.pdf)
but all codes are my own.
#### The goal of the game is to push all the boxes to the destinations 
![sokoban](https://user-images.githubusercontent.com/79690350/126941817-a7056730-caf3-40c7-babf-107763b746e0.jpeg)

#### First we import packages
#copy.deepcopy is needed for duplicating multidimensional lists in Python
```
import copy
from anytree import Node
```
#### Let's write functions to show the layout of the problem, get location of players and locations of boxes
```
def show_map(layout):
    for i in range(len(layout)):
        print(layout[i])
        
def location(layout,obj):
    line = [x for x in layout if obj in x][0]
    x_index = layout.index(line)
    y_index = line.index(obj)
    #print(x_index, y_index)
    return x_index, y_index

def locations(layout,obj):
    results = []
    for i in range(len(layout)):
        for j in range(len(layout[i])):
            if layout[i][j] == obj:
                results.append((i,j))
    return results
```
### Example problem is stored in a text file
#example problem
```
filename = "/Users/User/Downloads/problem.txt"
f = open(filename,"r")
lines=f.readlines()

example = []
for line in lines:
    example.append(list(line))
for i in example:
    i.remove("\n")
show_map(example)
```
![Screenshot 2021-07-26 at 12 01 25 PM](https://user-images.githubusercontent.com/79690350/126931394-1b4aeb30-582d-4751-b92c-2e714e827cc2.png)

### Change the layout accordingly after one move of up, down, left, or right (We only show the function for up here but all directions are basically the same function with slightly changed coordinates -- just think about how the game works)
```
def up(problem):
    layout = problem
    ##print("before:")
    #show_map(layout)
    player_on_destination = 0
    try:
        player = location(layout,"@")
    except: 
        player = location(layout,"+")
        player_on_destination = 1
    ##print(player)
    
    up = (player[0]-1,player[1])
    up_is_possible = 0

    
    up_next = layout.copy()
    
    #check if we can move up
    #if up is empty
    if layout[up[0]][up[1]] == " ":
        #print("up is empty")
        up_is_possible = 1
        
        #get the layout if we move up
        up_next[up[0]][up[1]] = "@"
        if player_on_destination == 1:
            up_next[player[0]][player[1]] = "."
        else:
            up_next[player[0]][player[1]] = " "
    #if up is box
    elif layout[up[0]][up[1]] == "$":
        #print("up is box")
        #if up up is empty
        if layout[up[0]-1][up[1]] == " ":
            up_is_possible = 1
            up_next[up[0]][up[1]] = "@"
            up_next[up[0]-1][up[1]] = "$"
            if player_on_destination == 1:
                up_next[player[0]][player[1]] = "."
            else:
                up_next[player[0]][player[1]] = " "
        #if up up is destination without box on it
        if layout[up[0]-1][up[1]] == ".":
            up_next[up[0]][up[1]] = "@"
            if player_on_destination == 1:
                up_next[player[0]][player[1]] = "."
            else:
                up_next[player[0]][player[1]] = " "
            up_next[up[0]-1][up[1]] = "*"
        #if up up is box or wall
        elif layout[up[0]-1][up[1]] == "$" or layout[up[0]-1][up[1]] == "#" or layout[up[0]-1][up[1]] == "*":
            up_is_possible = 0
            
    #if up is destination without box on it
    elif layout[up[0]][up[1]] == ".":
        #print("up is destination without box on it")
        up_is_possible = 1
        #get the layout if we move up
        up_next[up[0]][up[1]] = "+"
        if player_on_destination == 1:
            up_next[player[0]][player[1]] = "."
        else:
            up_next[player[0]][player[1]] = " "
        
    #if up is wall
    elif layout[up[0]][up[1]] == "#":
        pass
    #print("up is wall")
        
    #if up is destination with box on it 
    elif layout[up[0]][up[1]] == "*":
        #print("up is destination with box on it")
        #if up up is empty
        if layout[up[0]-1][up[1]] == " ":
            up_is_possible = 1
            up_next[up[0]][up[1]] = "+"
            up_next[up[0]-1][up[1]] = "$"
            if player_on_destination == 1:
                up_next[player[0]][player[1]] = "."
            else:
                up_next[player[0]][player[1]] = " "
        #if up up is destination without box on it
        if layout[up[0]-1][up[1]] == ".":
            up_is_possible = 1
            up_next[up[0]][up[1]] = "+"
            up_next[up[0]-1][up[1]] = "*"
            if player_on_destination == 1:
                up_next[player[0]][player[1]] = "."
            else:
                up_next[player[0]][player[1]] = " "
        #if up up is box or wall
        elif layout[up[0]-1][up[1]] == "$" or layout[up[0]-1][up[1]] == "#" or layout[up[0]-1][up[1]] == "*":
            up_is_possible = 0
    
    return up_next

#show_map(example)
example_copy = copy.deepcopy(example)
up(example_copy)
```
### Detect when the layout is in a deadlock and there's no way to reach destinations anymore
```
def corner(problem, box):
    layout = problem
    up = (box[0]-1,box[1])
    down = (box[0]+1,box[1])
    left = (box[0],box[1]-1)
    right = (box[0],box[1]+1)
    if (problem[up[0]][up[1]] == "#" and problem[left[0]][left[1]] == "#") or (problem[up[0]][up[1]] == "#" and problem[right[0]][right[1]] == "#"):
        return True
    elif (problem[down[0]][down[1]] == "#" and problem[left[0]][left[1]] == "#") or (problem[down[0]][down[1]] == "#" and problem[right[0]][right[1]] == "#"):
        return True
    else:
        return False
def wall(problem, box):
    layout = problem
    up = (box[0]-1,box[1])
    down = (box[0]+1,box[1])
    left = (box[0],box[1]-1)
    right = (box[0],box[1]+1)
    if (problem[up[0]][up[1]] == "#" and (problem[up[0]][up[1]-1] == "#" and problem[up[0]][up[1]+1] == "#")) or (problem[down[0]][down[1]] == "#" and (problem[down[0]][down[1]-1] == "#" and problem[down[0]][down[1]+1] == "#")):
        if "." in layout[box[0]]:
            return False
        else:
            #print("wall1")
            return True
        
    elif (problem[left[0]][left[1]] == "#" and (problem[left[0]-1][left[1]] == "#"and problem[left[0]+1][left[1]] == "#" )) or (problem[right[0]][right[1]] == "#" and (problem[right[0]-1][right[1]] == "#" and problem[right[0]+1][right[1]] == "#" )):
        #print("box[1]:",box[1])
        if "." in [row[box[1]] for row in layout]:
            return False
        else:
            #print("wall")
            return True
    else:
        return False
def deadlock(problem):
    layout = copy.deepcopy(problem)
    boxes = locations(layout,"$")
    for box in boxes:
        #print(box)
        if corner(layout,box) is True or wall(layout,box) is True:
            #print("deadlock") 
            return True
    return False
deadlock(example)
```
### Detect when the layout is a solution and all boxes are in the destinations
```
def solution(problem):
    layout = copy.deepcopy(problem)
    a = 0
    for i in layout:
        if "$" in i:
            return False
    return True
solution(up(example))
#example problem

filename = "/Users/James/Downloads/problem1.txt"
f = open(filename,"r")
lines=f.readlines()

example = []
for line in lines:
    example.append(list(line))
for i in example:
    i.remove("\n")
show_map(example)
```
### Get all possible next moves including the ones that result in a deadlock or not moving due to blockage (i.e. the "next move" can be the same as the previous move if for example moving up is not possible)
```
def next_possible(problem):
    layout = copy.deepcopy(problem)
    a = up(layout)
    layout = copy.deepcopy(problem)
    b = down(layout)
    layout = copy.deepcopy(problem)
    c = left(layout)
    layout = copy.deepcopy(problem)
    d = right(layout)
    return [a,b,c,d]
next_possible(example)
```
### Implementation of Breadth First Search using node tree to record path
```
visited = [] # List to keep track of visited nodes.
queue = []     #Initialize a queue

#Breadth First Search using node tree to record path
def bfs(visited, node):
    visited.append(node)
    queue.append(node)
    while queue:
        node = queue.pop(0)
        content = node.name
    
        
        #print ("s:",s, end = " ") 
        
        if solution(content) is True:
            print("solution reached")
            return content, node
        elif deadlock(content) is False:
            #print("not deadlocked")
            for child in next_possible(content):
                #print("neighbour:", neighbour)
                
                if child not in visited and deadlock(content) is False:
                    small = Node(child, parent=node)
                    visited.append(child)
                    queue.append(small)
                    
                    if solution(child) is True:
                        print("solution reached")
                        return child, small
        else:
            pass
            #print("deadlock encountered")
# Driver Code
e = Node(example, parent=None)
results = bfs(visited, e)
```
### Show the path to solving the Sokoban puzzle
```
for node in results[1].path:
    print(show_map(node.name))
```
![Screenshot 2021-07-26 at 12 44 37 PM](https://user-images.githubusercontent.com/79690350/126934198-ec282f72-fbeb-41eb-b672-0dc2f37f5e89.png)

