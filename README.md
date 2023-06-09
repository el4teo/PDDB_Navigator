~~~ txt
_________               __  ________    _________
\_   ___ \_____ _______|  | \__     \  /   _____/
/    \  \/\__  \\_  __ \  |  /   |   \ \_____  \ 
\     \____/ __ \|  | \/  |_/    |    \/        \
 \______  (____  /__|  |___/\_______  /_______  /
        \/     \/                   \/        \/ 
~~~

**CONTENT**
- [PD DB Navigator](#PD-DB-Navigator)
    - [Supported formats](#Supported-formats)
    - [Comparison file](#Comparison-file)
    - [Calling modes](#Calling-modes)

# PD DB Navigator

This project is intended to operate with a Partial Discharges (PD) Data Base (DB).

Some of its main functions are:
- Compare similitudes between defferent PD files
- Create new PD files from other files according to the user criteria
- Export PD files on different formats
- Remove PD events from the PD files
- Assig a waveform to each PD event (comming from a file format without waveform information)
- Export configurations of PD tests
- Export graphic representations of the PD files

##  Supported formats

The PD files that compose the DB must be saved on the local computer in one of the following formats:
- **CSV**:
    - Saved on ASCII format (more comprehensive)
    - 3 columns:
        - Time
        - Phase
        - Amplitude
    - Each row is a PD event
    - Doesn't have the waveform raw data
- **OMI**
    - Saved on binary format (more efficient)
    - Composed of two files `.Q` and `.PH`
        - `.Q` saves the `time` and `charge` of the pulses
        - `.PH` saves the `phase angle` of the pulses
        - Both files must be on the same directory without any other file or directory
    - Doesn't have the waveform raw data
- **BIN**
    - Saved on binary format (more efficient)
    - Stores each samble as a `short`
    - It can contain the waveform raw data
    - It's size is too high

### CSV - Reading and writing
### OMI - Reading and writing
### BIN - Reading and writing

## Comparison file

During a patterns comparison the idea is to see how similar is each defect with the rest of the deffects of the same pattern.

Some formulas have been created to navigate and interpratate the results file.

### n_comps(n_files)
The number of required comparisons:
~~~ C++
n_comps = n_files * (n_files - 1) / 2;
~~~

For example, with a small set of four files the posible comparisions are:
~~~ txt
f1 vs f2
f1 vs f3
f1 vs f4
f2 vs f3
f2 vs f4
f3 vs f4
~~~

As the idea is to make a huge amount of comparisons and there is quadratic member in the formula:
- The results file is as small as posible:
    - It only saves the similitude indexes
    - Each index is a `float` number
    - It is saved on a binary format
    - Each `float` is made of 4 Bytes
    - The size of the results file (in bytes) revels the number of compared files

For example, if a deffect is composed by 1000:
~~~ C++
n_files = 1000;
n_comps = n_files * (n_files - 1) / 2;
// n_comps = 500'500
file_size_Bytes = n_comps * 4;
// file_size_Bytes = 2'002'000
// file_size_kB = 2'002
// file_size_MB = 2.002
~~~

### n_files(file_size_Bytes)
The number of files compared based on the results file size:
~~~ C++
n_comps = file_size_Bytes / 4;
// n_comps = n_files * (n_files - 1) / 2;
// 2 * n_comps =  n_files^2 - n_files;
// n_files^2 - n_files - 2 * n_comps = 0;
// ax^2 + bx + c = 0
// x1 = (-b + sqrt (b * b - 4 * a * c)) / (2 * a); 

n_files = (1 + sqrt(1 + 8 * n_comps))/2;
n_files = (1 + sqrt(1 + 2 * file_size_Bytes))/2;
~~~

### firstTimeF1(n_comps, idxF1)
NOTE: Take in consideration that for this purpose:
- The indexes of the files start on 1
- The index of the array starts on 0

As the array represents comparison results, each number is related to two files.

Let's call the file of the lower index as `F1`, and the file of the higher indes `F2`

First of all the first index with F1 must be calculated (fIdx):

~~~ C++
// Get n_comps in one of the following ways
n_comps = file_size_Bytes / 4;
n_comps = n_files * (n_files - 1) / 2;

// First index of F1
firstTimeF1 = n_comps - ((n_files-idxF1) * (n_files-idxF1 + 1) / 2);
~~~

An example of 5 files is presented to show the concept:

| ArrIdx| idxF1 | idxF2 |
|:-----:|:-----:|:-----:|
| 0 | 1 | 2 |
| 1 | 1 | 3 |
| 2 | 1 | 4 |
| 3 | 1 | 5 |
| 4 | 2 | 3 |
| 5 | 2 | 4 |
| 6 | 2 | 5 |
| 7 | 3 | 4 |
| 8 | 3 | 5 |
| 9 | 4 | 5 |

On the next table you can see how the formula works:

| idxF1 | firstTimeF1 |
|:-----:|:-----|
| 1 | 0 = 10 - 10|
| 2 | 4 = 10 - 6 |
| 3 | 7 = 10 - 3 |
| 4 | 9 = 10 - 1 |
| 5 | (last never F1) |

### idxInArr(n_comps, idxF1, idxF2)
NOTE: Take in consideration that for this purpose:
- The indexes of the files start on 1
- The index of the array start on 0

Fianlly, to get the index inside the array of the desired comparison (idxInArr):
~~~ C++
// Get n_comps in one of the following ways
n_comps = file_size_Bytes / 4;
n_comps = n_files * (n_files - 1) / 2;

// First index of F1
firstTimeF1 = n_comps - (n_files-idxF1) * (n_files-idxF1 + 1) / 2;

// Index of comparison F1 vs F2
idxInArr = firstTimeF1 + idxF2 - idxF1 - 1;
~~~

The next table shows the concept:

| ArrIdx| Comparison | firstTimeF1 | idxInArr|
|:-----:|:----------:|:-----------:|:-------:|
| 0 | 1 vs 2 | 0 | 0 + 2 - 1 - 1 |
| 1 | 1 vs 3 | 0 | 0 + 3 - 1 - 1 |
| 2 | 1 vs 4 | 0 | 0 + 4 - 1 - 1 |
| 3 | 1 vs 5 | 0 | 0 + 5 - 1 - 1 |
| 4 | 2 vs 3 | 4 | 4 + 3 - 2 - 1 |
| 5 | 2 vs 4 | 4 | 4 + 4 - 2 - 1 |
| 6 | 2 vs 5 | 4 | 4 + 5 - 2 - 1 |
| 7 | 3 vs 4 | 7 | 7 + 4 - 3 - 1 |
| 8 | 3 vs 5 | 7 | 7 + 5 - 3 - 1 |
| 9 | 4 vs 5 | 9 | 9 + 5 - 4 - 1 |

### Using the comparison file

In this section you will find a couple of programming examples of how to read and use the comparison file (___*.pdcmp*___).

#### C++
~~~ C++
#include <iostream>
#include <fstream>
#include <filesystem>

using namespace std;
namespace fs = std::filesystem;

vector<float> read_pdcmp(string& orig_file) {
	// Open the file in binary mode
	ifstream file(orig_file, ios::binary);

	if (!file) {
		throw runtime_error("Imposible to open " + orig_file);
	}

	// Calculate number of readings
	long file_size_Bytes = fs::file_size(orig_file);
	long n_comps = file_size_Bytes / 4;
	vector<float> vec(n_comps);

	// Read the number from the file
	for (long i = 0; i < n_comps; i++) {
		file.read(reinterpret_cast<char*>(&vec[i]), sizeof(float));
		if (!file) {
			file.close();
			throw runtime_error("Error while reading " + orig_file);
		}
	}

	// Close the file
	file.close();

	return vec;
}
~~~

#### MATLAB
~~~ matlab
% Open the file in binary mode
fileID = fopen('data.pdcmp', 'rb');

if fileID == -1
    error('Failed to open the file.');
end

% Get the file size
fseek(fileID, 0, 'eof');
file_size_Bytes = ftell(fileID);
fseek(fileID, 0, 'bof');

% Calculate number of readings
n_comps = file_size_Bytes / 4;
vec = zeros(n_comps, 1);

for i = 1:n_comps
    % Read the number from the file
    vec(i) = fread(fileID, 1, 'float');
end

% Close the file
fclose(fileID);
~~~

## Calling modes

Option 1: With the path of the database as argument

~~~
Reduced_DB.exe [path_db]
~~~

Option 2: With the path of the database in an auxiliary file on following path relative to the `.exe` file:

`memory\DEF_DB_PATH.txt`

~~~
Reduced_DB.exe
~~~

### Functionalities

#### Comparation

This will export a folder with the `.pdcmp` files of the selected deffects.

By default all the deffects are selected, if you want a custom quantity use the `Set selected defects` option

Steps:
- [3]: Open DB
- [5]: Compare patterns [...]
  - [2]: Set selected defects (optional)
  - [3]: Exhaustive comparison

#### Export def. names

As the `.pdcmp` files only save the results of the comparison of different files, this functionality may be needed in order to pass from a index to a filename.

The program will generate `.names` files for the selecetd defects. The idea is to use the same selection from the [comparation](#comparation)
Steps:
- [3]: Open DB
- [5]: Compare patterns [...]
  - [4]: Export selected PD file names


#### Export XLSX

[used lib](https://libxlsxwriter.github.io/getting_started.html)

This functionality will convert the `.pdcmp` files to a `.xlsx` files, more easy to interpret from a human point of view.

One sheet contains the matrix, and an additional one contains useful info.
