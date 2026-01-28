#include <iostream>
#include <vector>
#include <fstream>
#include <string>

using namespace std;

struct Reading {
    string type; // BS, IS, FS
    double staffReading;
    string stationName;
};

class InstrumentSetup {
private:
    int setupID;
    double HOC;
    vector<Reading> readings;

public:
    InstrumentSetup(int id, double hoc) : setupID(id), HOC(hoc) {}

    void add_reading(const Reading& reading) {
        readings.push_back(reading);
    }

    vector<double> calculate_RL(double initialRL) {
        vector<double> RLs;
        double currentRL = initialRL;
        for (const auto& r : readings) {
            if (r.type == "BS") {
                currentRL = HOC - r.staffReading;
            } else if (r.type == "FS") {
                HOC = currentRL + r.staffReading;
            }
            RLs.push_back(currentRL);
        }
        return RLs;
    }

    void display_level_book() {
        cout << "Setup ID: " << setupID << ", HOC: " << HOC << endl;
        for (const auto& r : readings) {
            cout << r.type << " " << r.staffReading << " at " << r.stationName << endl;
        }
    }

    void export_to_file(const string& filename) {
        ofstream file(filename);
        file << "Setup ID,HOC,Reading Type,Staff Reading,Station\n";
        for (const auto& r : readings) {
            file << setupID << "," << HOC << "," << r.type << "," << r.staffReading << "," << r.stationName << "\n";
        }
        file.close();
    }
};

int main() {
    InstrumentSetup setup(1, 100.0);

    Reading r1 = {"BS", 1.5, "A"};
    Reading r2 = {"IS", 2.0, "B"};
    Reading r3 = {"FS", 1.2, "C"};

    setup.add_reading(r1);
    setup.add_reading(r2);
    setup.add_reading(r3);

    setup.display_level_book();
    auto RLs = setup.calculate_RL(95.0);
    cout << "Reduced Levels: ";
    for (auto rl : RLs) cout << rl << " ";
    cout << endl;

    setup.export_to_file("leveling_book.csv");
    return 0;
}
