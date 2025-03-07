#include <iostream>
#include <fstream>
#include <string>

using namespace std;

const int MAX_SIZE = 1000; // Μέγιστος αριθμός στοιχείων

class BibElement {
public:
    string type, id, author, title, journal, volume, booktitle, publisher, pages, year;

    BibElement() {} // Προεπιλεγμένος κατασκευαστής

    BibElement(string t, string i, string a, string ti, string j, string v, string b, string p, string pa, string y)
        : type(t), id(i), author(a), title(ti), journal(j), volume(v), booktitle(b), publisher(p), pages(pa), year(y) {}

    void writeToFile(ofstream& outFile) const 
	{
        outFile << type << "\n" << id << "\n" << author << "\n" << title << "\n"
                << journal << "\n" << volume << "\n" << booktitle << "\n"
                << publisher << "\n" << pages << "\n" << year << "\n\n";
    }
};

// Διαβάζει βιβλιογραφικές αναφορές από αρχείο και τις αποθηκεύει στον πίνακα
int readBibFile(const string& filename, BibElement elements[], int& count) 
{
    ifstream inputFile(filename);
    count = 0;

    if (!inputFile.is_open()) {
        cerr << "Error opening file: " << filename << endl;
        return -1;
    }

    string line, type, id, author, title, journal, volume, booktitle, publisher, pages, year;
    while (getline(inputFile, line)) {
        if (line.empty()) continue;

        if (line.find("@") == 0) { // Νέα βιβλιογραφική αναφορά
            type = line;
            getline(inputFile, id);
            getline(inputFile, author);
            getline(inputFile, title);
            getline(inputFile, journal);
            getline(inputFile, volume);
            getline(inputFile, booktitle);
            getline(inputFile, publisher);
            getline(inputFile, pages);
            getline(inputFile, year);

            if (count < MAX_SIZE) {
                elements[count++] = BibElement(type, id, author, title, journal, volume, booktitle, publisher, pages, year);
            } else {
                cerr << "Error: Maximum bibliography size exceeded!" << endl;
                break;
            }
        }
    }

    inputFile.close();
    return count;
}


// Σύγκριση δύο BibElement με βάση τα κριτήρια
bool compareBibElements(const BibElement& a, const BibElement& b, const string& primary, const string& secondary) 
{
    string primaryA, primaryB, secondaryA, secondaryB;

    if (primary == "type") { primaryA = a.type; primaryB = b.type; }
    else if (primary == "id") { primaryA = a.id; primaryB = b.id; }
    else if (primary == "author") { primaryA = a.author; primaryB = b.author; }
    else if (primary == "year") { primaryA = a.year; primaryB = b.year; }

    if (secondary == "type") { secondaryA = a.type; secondaryB = b.type; }
    else if (secondary == "id") { secondaryA = a.id; secondaryB = b.id; }
    else if (secondary == "author") { secondaryA = a.author; secondaryB = b.author; }
    else if (secondary == "year") { secondaryA = a.year; secondaryB = b.year; }

    if (primaryA != primaryB) return primaryA < primaryB; // Αύξουσα ταξινόμηση
    return secondaryA < secondaryB;
}

// Ταξινόμηση πίνακα με Bubble Sort
void sortBibElements(BibElement elements[], int count, const string& primary, const string& secondary) 
{
    for (int i = 0; i < count - 1; i++) {
        for (int j = 0; j < count - i - 1; j++) {
            if (compareBibElements(elements[j], elements[j + 1], primary, secondary)) {
                swap(elements[j], elements[j + 1]);
            }
        }
    }
}

// Εγγραφή των ταξινομημένων στοιχείων σε αρχείο
void writeBibToFile(const string& filename, BibElement elements[], int count) 
{
    ofstream outFile(filename);
    if (!outFile.is_open()) 
	{
        cerr << "Error opening output file!" << endl;
        return;
    }

    for (int i = 0; i < count; i++) 
	{
        elements[i].writeToFile(outFile);
    }

    outFile.close();
    cout << "Bibliography elements have been written to " << filename << endl;
}

int main() 
{
    string inputFilename;
    cout << "Enter the input file name: ";
    cin >> inputFilename;

    BibElement bibElements[MAX_SIZE];
    int count = 0;

    if (readBibFile(inputFilename, bibElements, count) == -1) {
        return 1;
    }

    string primaryCriteria, secondaryCriteria;
    cout << "Enter primary sorting criterion (type, id, author, year): ";
    cin >> primaryCriteria;
    cout << "Enter secondary sorting criterion (type, id, author, year): ";
    cin >> secondaryCriteria;

    sortBibElements(bibElements, count, primaryCriteria, secondaryCriteria);
    writeBibToFile("output.txt", bibElements, count);

    return 0;
}
