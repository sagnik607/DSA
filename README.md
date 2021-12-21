# DSA
RED-BLACK TREE
#include <stdio.h> 
#include <stdlib.h>

// Define Bool
typedef enum Bool {
    false, true
} Bool;

// Define Color
typedef enum Color {
    red, black
} Color; 

// Define Node
typedef struct Node { 
    int data; 
    Color color; 
    struct Node *leftChild, *rightChild, *parent; 
} Node; 

// Create New Node
Node *createNewNode(int value){
    Node* temp = (Node*)malloc(sizeof(Node));
	temp -> parent = NULL;
    temp -> leftChild = NULL;
    temp -> rightChild = NULL; 
	temp -> color = red;
    temp -> data = value;
    return temp;
} 
 
// Search Node
Node *search(Node *root, int value) { 
	Node *temp = root; 
	
    while (temp != NULL) { 
	    if (value < temp -> data) { 
		    if (temp -> leftChild == NULL) { 
		        break; 
            } else {
		        temp = temp -> leftChild; 
            }
        } else if (value == temp -> data) { 
            break; 
        } else { 
            if (temp -> rightChild == NULL) { 
                break; 
            } else {
                temp = temp -> rightChild;
            } 
        } 
	} 

	return temp; 
} 

// Check whether it is left child or not
Bool isOnLeft(Node* node) { 
    return node == node -> parent -> leftChild; 
} 

// Ge uncle node
Node *uncleNode(Node* node) { 
	if (node->parent == NULL || node->parent->parent == NULL) 
	return NULL; 

	if (isOnLeft(node->parent)) {
	    return node -> parent -> parent -> rightChild; 
    } else {
	    return node -> parent -> parent -> leftChild;
    } 
}

// Swap colors of two nodes
void swapColors(Node *firstNode, Node *secondNode) { 
	Color temp; 
	temp = firstNode -> color; 
	firstNode -> color = secondNode -> color; 
	secondNode -> color = temp; 
} 

// Move parent node above given node
void moveDown(Node *node, Node *newParent) { 
	if (node -> parent != NULL) { 
        if (isOnLeft(node)) { 
            node -> parent -> leftChild = newParent; 
        } else { 
            node -> parent -> rightChild = newParent; 
        } 
	} 
	newParent -> parent = node -> parent; 
	node -> parent = newParent; 
} 

// Left Rotate
void leftRotate(Node **root, Node *node) { 
	// new parent will be node's right child 
	Node *newParent = node -> rightChild; 

	if (node == *root) {
	    *root = newParent; 
    }

	moveDown(node,newParent); 

	node -> rightChild = newParent -> leftChild; 
	
	if (newParent -> leftChild != NULL) { 
	    newParent -> leftChild -> parent = node; 
    }

	newParent -> leftChild = node; 
} 

// Right Rotate
void rightRotate(Node **root, Node *node) { 
	Node *newParent = node -> leftChild; 

	if (node == *root) { 
	    *root = newParent; 
    }

	moveDown(node,newParent); 

	node -> leftChild = newParent -> rightChild; 
	
	if (newParent -> rightChild != NULL) { 
	    newParent -> rightChild -> parent = node; 
    }

	newParent -> rightChild = node; 
} 

// Fix red red violation 
void fixRedRed(Node **root, Node *node) { 
	if (node == *root) { 
        node -> color = black; 
        return; 
	} 

	Node *parent = node -> parent, *grandparent = parent -> parent, *uncle = uncleNode(node); 

	if (parent -> color != black) { 
        if (uncle != NULL && uncle -> color == red) {  
            parent -> color = black; 
            uncle -> color = black; 
            grandparent -> color = red; 
            fixRedRed(root, grandparent); 
        } else {  
            if (isOnLeft(parent)) { 
                if (isOnLeft(node)) { 
                    // Left Right 
                    swapColors(parent, grandparent); 
                } else { 
                    leftRotate(root, parent); 
                    swapColors(node, grandparent); 
                } 
                rightRotate(root, grandparent); 
            } else { 
                if (isOnLeft(node)) { 
                    // Right Left
                    rightRotate(root, parent); 
                    swapColors(node, grandparent); 
                } else { 
                    swapColors(parent, grandparent); 
                } 
 
                leftRotate(root, grandparent); 
            } 
        } 
	} 
} 

// Insert into tree 
void insert(Node **root, int value) { 
	Node *newNode = createNewNode(value);

	if (*root == NULL) { 
        newNode -> color = black; 
        *root = newNode; 
	} else { 
	    Node *temp = search(*root, value); 

	    if (temp -> data == value) { 
            printf("/nDuplicate Entry:\t%d\n", value);
		    return; 
	    } 

	    newNode -> parent = temp; 

        if (value < temp -> data) { 
            temp -> leftChild = newNode; 
        } else {
            temp -> rightChild = newNode;
        } 
         
        fixRedRed(root, newNode); 
	} 
} 

// Find Predecessor 
Node *predecessor(Node *node) { 
	Node *temp = node; 

	while (temp -> rightChild != NULL) { 
	    temp = temp -> rightChild; 
    }

	return temp; 
} 

// Find node to replace the deleted node
Node *BSTreplace(Node *node) {  
	if (node -> leftChild != NULL && node -> rightChild != NULL) { 
	    return predecessor(node -> leftChild); 
    }
 
	if (node -> leftChild == NULL && node -> rightChild == NULL) { 
	    return NULL; 
    }

	if (node -> leftChild != NULL) { 
	    return node -> leftChild; 
    } else {
	    return node -> rightChild;
    } 
} 

// Get Sibling 
Node *siblingNode(Node* node) { 
	if (node -> parent == NULL) {
	    return NULL; 
    }

	if (isOnLeft(node)) {
	    return node -> parent -> rightChild; 
    }

	return node -> parent -> leftChild; 
} 

// Check whether node has a red child
Bool hasRedChild(Node* node) { 
	return (node -> leftChild != NULL && node -> leftChild -> color == red) || (node -> rightChild != NULL && node -> rightChild -> color == red);
}

// Fix DoubleBlack
void fixDoubleBlack(Node **root, Node *node) { 
	if (node == *root) {
	    return; 
    }

	Node *sibling = siblingNode(node), *parent = node->parent; 

	if (sibling == NULL) {  
	    fixDoubleBlack(root, parent); 
	} else { 
        if (sibling -> color == red) { 
            parent -> color = red; 
            sibling -> color = black; 
            if (isOnLeft(sibling)) { 
                // Left Left 
                rightRotate(root, parent); 
            } else { 
                // Right Right 
                leftRotate(root, parent); 
            } 
            fixDoubleBlack(root, node); 
        } else { 
            if (hasRedChild(sibling)) { 
                if (sibling -> leftChild != NULL && sibling -> leftChild -> color == red) { 
                    if (isOnLeft(sibling)) { 
                        // Left Left
                        sibling -> leftChild -> color = sibling -> color; 
                        sibling -> color = parent -> color; 
                        rightRotate(root, parent); 
                    } else { 
                        // Right Left 
                        sibling -> leftChild -> color = parent -> color; 
                        rightRotate(root, sibling); 
                        leftRotate(root, parent); 
                    } 
                } else { 
                    if (isOnLeft(sibling)) { 
                        // Left Right
                        sibling -> rightChild -> color = parent -> color; 
                        leftRotate(root, sibling); 
                        rightRotate(root, parent); 
                    } else { 
                        // right right 
                        sibling -> rightChild -> color = sibling -> color; 
                        sibling -> color = parent -> color; 
                        leftRotate(root, parent); 
                    } 
                } 
                parent -> color = black; 
            } else {  
                sibling -> color = red; 
                if (parent -> color == black) { 
                    fixDoubleBlack(root, parent);
                } else {
                    parent -> color = black;
                } 
            } 
        } 
        
    } 
} 

// Swap Values
void swapValues(Node *node1, Node *node2) { 
	int temp; 
	temp = node1 -> data; 
	node1 -> data = node2 -> data; 
	node2 -> data = temp; 
}

// Delete Node
void deleteNode(Node **root, Node *node) { 
	Node *replacedNode = BSTreplace(node); 

	// Check whether replaced and node are both black
	Bool bothBlack = ((replacedNode == NULL || replacedNode -> color == black) && node -> color == black); 
	Node *parent = node -> parent; 
    
    // If Leaf
	if (replacedNode == NULL) { 
        if (node == *root) { 
            *root = NULL; 
        } else { 
            if (bothBlack) { 
                fixDoubleBlack(root, node); 
            } else { 
                if (siblingNode(node) != NULL) {  
                    siblingNode(node) -> color = red;
                } 
            } 

            if (isOnLeft(node)) { 
                parent -> leftChild = NULL; 
            } else { 
                parent -> rightChild = NULL; 
            } 
        } 
        free(node); 
        return; 
	} 

	if (node -> leftChild == NULL || node -> rightChild == NULL) { 
        if (node == *root) { 
            node -> data = replacedNode -> data; 
            node -> leftChild = node -> rightChild = NULL; 
            free(replacedNode); 
        } else {  
            if (isOnLeft(node)) { 
                parent -> leftChild = replacedNode; 
            } else { 
                parent -> rightChild = replacedNode; 
            } 
            
            free(node); 
            replacedNode -> parent = parent; 
            
            if (bothBlack) {  
                fixDoubleBlack(root, replacedNode); 
            } else {  
                replacedNode -> color = black; 
            } 
        } 
        return; 
	} 

	swapValues(replacedNode, node); 
	deleteNode(root, replacedNode); 
} 

// Delete the given value 
void deleteByVal(Node **root, int value) { 
	if (*root == NULL) { 
	    return;
    } 

	Node *node = search(*root, value); 
    // Node *u;

	if (node -> data != value) { 
	    printf("No node found to delete with value: %d\n", value); 
	    return; 
	} 

	deleteNode(root, node); 
}  

//Get PreOrder
void preOrder(Node *root) { 
	if (root == NULL) { 
	    return;
    }
	printf(" %d ", root -> data); 
	preOrder(root -> leftChild); 
	preOrder(root -> rightChild); 
} 

//Get InOrder
void inOrder(Node *root) { 
	if (root == NULL) { 
	    return;
    }
	inOrder(root -> leftChild); 
	printf(" %d ", root -> data); 
	inOrder(root -> rightChild); 
} 

//Get PostOrder
void postOrder(Node *root) { 
	if (root == NULL) { 
	    return;
    }
	postOrder(root -> leftChild); 
	postOrder(root -> rightChild); 
	printf(" %d ", root -> data); 
} 

// Print PreOrder 
void printPreOrder(Node* root) { 
	printf("Preorder: "); 
	if (root == NULL) { 
	    printf("Tree is empty \n");
    } else {
	    preOrder(root);
    } 
	printf("\n"); 
} 

// Print InOrder 
void printInOrder(Node* root) { 
	printf("Inorder: "); 
	if (root == NULL) { 
	    printf("Tree is empty \n");
    } else {
	    inOrder(root);
    } 
	printf("\n"); 
} 

// Print PostOrder
void printPostOrder(Node* root) { 
	printf("Postorder: "); 
	if (root == NULL) { 
	    printf("Tree is empty \n");
    } else {
	    postOrder(root);
    } 
	printf("\n"); 
} 

// Main Program
int main() {
    Node *root = NULL;

    int choice, initial_size, value;

    printf("\n----------------------------------------------");
    printf("\nSubject Name:\tData Structures and Algorithms");
    printf("\nYour Name:\tSAGNIK PAHARI");
    printf("\nRoll Number:\t002111002002");
    printf("\nProgram:\tRed Black Tree");
    printf("\n----------------------------------------------");
    
    do {
        printf("\n\nMenu");
        printf("\n-----");
        printf("\n1) Insert");
        printf("\n2) Delete");
        printf("\n3) View");
        printf("\n4) Exit");

        printf("\n\nEnter a choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                printf("\nEnter the number of values to insert: ");
                scanf("%d", &initial_size);

                printf("\nEnter the values (no duplicates): ");
                for(int i = 0; i < initial_size; i++) {
                    scanf("%d", &value);
                    insert(&root, value);
                }
                printInOrder(root);
                break;
            case 2:
                printf("\nEnter the number of values to delete: ");
                scanf("%d", &initial_size);

                printf("\nEnter the values (no duplicates): ");
                for(int i = 0; i < initial_size; i++) {
                    scanf("%d", &value);
                    deleteByVal(&root, value);
                }
                printInOrder(root);
                break;
            case 3:
                printPreOrder(root);
                printInOrder(root);
                printPostOrder(root);
                break;
            case 4:
                printf("\n");
                break;
            default:
                printf("\nInvalid choice!!!");
                break;
        }
    } while (choice != 4);
    
    return (0);
}
