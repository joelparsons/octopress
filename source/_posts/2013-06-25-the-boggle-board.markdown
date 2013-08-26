---
layout: post
title: "The Boggle Board Question"
date: 2013-06-25 14:12
comments: false
categories: 
---

So I was looking into the "Boggle Board" question which is posted on several programming interview sites. The question is pretty straight forward even if you dont know the game "boggle" which it is based on. You have a nxm matrix of letters and the aim of the game is to make words from those letters. From any arbitraty starting letter you are allowed to travel from letter to letter in any direction including diagonals with the restriction that you cant use the same letter twice. 

The interview quetion usually asks candidates to to determine, given a 4x4 boggle board, whether a given word is in the board.

The problem seems deceptively simple but the requirement that you mustnt revisit a letter means that any kind of recursive solution is going to have to keep track of a whole bunch of state and get messy really quickly.

## Graph Search

The simplicity of a graph search based solution becomes apparent if you solve the meaty part of the problem first.

To solve this problem with a graph search algorithm we need the letters in a graph. Lets create a node object that we can build a graph out of:

{% codeblock lang:objc %}
@interface Node : NSObject
@property (nonatomic, strong) NSString * letter;
@property (nonatomic, strong) NSMutableSet * adjacentNodes;
@end
{% endcodeblock %}

We would create a `Node \*` for each letter in the boggle board and then link it to the nodes that letter is adjacent to. There is now no need to store the Nodes in an array matrix that represents the board or anything like that as all the information about the board we need is encapsulated in these two pieces of data. Now assuming we can get the board represented in a bunch of `Node` objects all in an `NSSet` the algorithm for searching through them is relatively simple.

We take each `Node` in turn and do a depth first search on its neighbours for a match for the next letter in the word we are checking for.

{% codeblock lang:objc %}
BOOL searchForWord((NSString \*)word, NSSet * nodeGraph){
    NSString * firstLetter = [NSString stringWithFormat:@"%c",[word characterAtIndex:0]];
    for (Node * node in nodeGraph){
        NSMutableSet * visitedNodes = [[NSMutableSet alloc] initWithCapacity:word.length];
        BOOL found = depthFirstSearch(node, word, 0, visitedNodes);
        if (found) return found;
    }

    return NO;
}
{% endcodeblock %}

For the `depthFirstSearch` function we need to do a few things.
1. Keep track of visited Nodes so we don't visit the same one twice
2. Keep track of depth so we know our position in the word
3. Recursively search through the adjacent nodes

{%codeblock lang:objc %}
BOOL depthFirstSearch(Node * node, NSString * word, NSInteger depth, NSMutableSet * visitedNodes){
 
    unichar nodeLetter = [node.letter characterAtIndex:0];
    unichar wordLetter = [word characterAtIndex:depth];
 
    [visitedNodes addObject:node];
 
    if (nodeLetter == wordLetter){
        //if we have a match and have letters left keep searching
        //else we are at the end of the word so return YES
        if (depth < word.length - 1) {
            for (Node * adjacentNode in node.adjacentNodes){
                if ([visitedNodes containsObject:adjacentNode]) continue;
                BOOL found = depthFirstSearch(adjacentNode, word, depth + 1, visitedNodes);
                if (found) return found;
            }
        }
        else return YES;
    }
 
    [visitedNodes removeObject:node];
    return NO;
}
{% endcodeblock %}

And thats it. Two simple functions achieve the objectives of the questions in 30 lines of code. For some interviewers this may be enough as this is considered to be the hard part of the question.

## Going Further

There are several directions you could take this to make it more fully formed. For example, given a matrix of arrays could we build the graph? Also if were keeping the graph around it might make sense to make a graph object and make the search a method on a graph instance along with a graph generation method on the class. Ive tied it all together in a excerpt you can actually paste and run into an xcode command line app.

{% codeblock main.m  https://gist.github.com/joelparsons/5666671 Github Gist %}
#import <Foundation/Foundation.h>
 
@interface Node : NSObject
@property (nonatomic, strong) NSString * letter;
@property (nonatomic, strong) NSMutableSet * adjacentNodes;
@end
 
@implementation Node
-(NSMutableSet *)adjacentNodes{
    if(_adjacentNodes){
        return _adjacentNodes;
    }
    _adjacentNodes = [[NSMutableSet alloc] init];
    return _adjacentNodes;
}
@end
 
 
@interface Graph : NSObject
@property (nonatomic, strong) NSSet * nodes;
@property (nonatomic, strong) NSDictionary * lookupDictionary;
 
+(instancetype)graphForBoggleBoard:(NSArray *)board;
 
-(BOOL)searchForWord:(NSString *)word;
@end
 
@implementation Graph
 
+(instancetype)graphForBoggleBoard:(NSArray *)board{
    NSMutableSet * nodes = [[NSMutableSet alloc] init];
    NSMutableDictionary * dictionary = [NSMutableDictionary dictionary];
 
    NSMutableArray * nodeBoard = [[NSMutableArray alloc] initWithCapacity:board.count];
    for (NSArray * row in board){
        NSMutableArray * nodeRow = [[NSMutableArray alloc] initWithCapacity:row.count];
        for (NSString * letter in row){
            Node * node = [[Node alloc] init];
            node.letter = letter;
            
            [nodeRow addObject:node];
            [nodes addObject:node];
            if (dictionary[node.letter]){
                [dictionary[node.letter] addObject:node];
            }else{
                dictionary[node.letter] = @[node].mutableCopy;
            }
        }
        [nodeBoard addObject:nodeRow];
    }
 
 
    NSArray * previousRow = nil;
    for (NSInteger row = 0; row < nodeBoard.count; row++){
        NSArray * nodeRow = nodeBoard[row];
        for (NSInteger col = 0; col < nodeRow.count; col ++){
            Node * currentNode = nodeBoard[row][col];
            for (int i = col - 1; i <= col + 1; i++){
                if (i < 0 || i >= nodeRow.count)
                    continue;
                Node * node = previousRow[i];
                if (node){
                    [currentNode.adjacentNodes addObject:node];
                    [node.adjacentNodes addObject:currentNode];   
                }
            }
            if (col > 0){
                Node * node = nodeBoard[row][col - 1];
                [currentNode.adjacentNodes addObject:node];
                [node.adjacentNodes addObject:currentNode];
            }
        }
        previousRow = nodeRow;
    }
 
 
    Graph * graph = [[Graph alloc] init];
    graph.lookupDictionary = dictionary;
    graph.nodes = nodes;
    return graph;
}
 
BOOL depthFirstSearch(Node * node, NSString * word, NSInteger depth, NSMutableSet * visitedNodes){
 
    unichar nodeLetter = [node.letter characterAtIndex:0];
    unichar wordLetter = [word characterAtIndex:depth];
 
    [visitedNodes addObject:node];
 
    if (nodeLetter == wordLetter){
        if (depth < word.length - 1) {
            for (Node * adjacentNode in node.adjacentNodes){
                if ([visitedNodes containsObject:adjacentNode]) continue;
                BOOL found = depthFirstSearch(adjacentNode, word, depth + 1, visitedNodes);
                if (found) return found;
            }
        }
        else return YES;
    }
 
    [visitedNodes removeObject:node];
    return NO;
}
 
-(BOOL)searchForWord:(NSString *)word{
    NSString * firstLetter = [NSString stringWithFormat:@"%c",[word characterAtIndex:0]];
    NSSet * firstNodes = self.lookupDictionary[firstLetter];
    for (Node * node in firstNodes){
        NSMutableSet * visitedNodes = [[NSMutableSet alloc] initWithCapacity:word.length];
        BOOL found = depthFirstSearch(node, word, 0, visitedNodes);
        if (found) return found;
    }
 
    return NO;
}
@end
 
 
int main(int argc, char *argv[]) {
    @autoreleasepool {
        NSArray * boggleBoard = @[
                                  @[@"a", @"b", @"c"],
                                  @[@"d", @"d", @"h"],
                                  @[@"e", @"i", @"u"]
                                  ];
 
        Graph * graph = [Graph graphForBoggleBoard:boggleBoard];
        NSLog (@"Word %@", [graph searchForWord:@"chuddd"] ? @"exists" : @"doesnt exist");
    }
    return 0;
}
{% endcodeblock %}
