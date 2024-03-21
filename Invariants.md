```C++
#include <iostream>
#include <cassert>

class List {
private:
    struct Node {
        int data;
        Node* next;
        Node(int value, Node* nextNode = nullptr) : data(value), next(nextNode) {}
        ~Node() { delete next; }
    };

    Node* theList = nullptr;

public:
    ~List() { delete theList; }

    void add(int value) {
        theList = new Node(value, theList);
    }

    int ith(int index) const {
        Node* current = theList;
        for (int i = 0; i < index; ++i) {
            assert(current != nullptr && "Index out of bounds");
            current = current->next;
        }
        assert(current != nullptr && "Index out of bounds");
        return current->data;
    }

    class Iterator {
    private:
        Node* ptr;

    public:
        Iterator(Node* node) : ptr(node) {}

        int& operator*() const {
            assert(ptr != nullptr && "Dereferencing null pointer");
            return ptr->data;
        }

        Iterator& operator++() {
            assert(ptr != nullptr && "Incrementing null pointer");
            ptr = ptr->next;
            return *this;
        }

        bool operator==(const Iterator& other) const {
            return ptr == other.ptr;
        }

        bool operator!=(const Iterator& other) const {
            return !(*this == other);
        }
    };

    Iterator begin() const {
        return Iterator(theList);
    }

    Iterator end() const {
        return Iterator(nullptr);
    }
};

int main() {
    List myList;
    myList.add(1);
    myList.add(3);
    myList.add(5);

    // Using the iterator to traverse the list
    for (auto it = myList.begin(); it != myList.end(); ++it) {
        std::cout << *it << ' ';
    }
    std::cout << std::endl;

    return 0;
}

```