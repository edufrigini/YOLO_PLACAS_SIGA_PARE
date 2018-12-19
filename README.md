# PASSO A PASSO COMO TREINAR A YOLO PARA DETECTAR PLACAS DE SIGA E PARE
# Autores: Eduardo Frigini e Deivison Vitoria

ANTES DE MAIS NADA EH NECESSARIO TIRAR VARIAS FOTOS DE PLACAS DE SIGA E PARE. TIRAMOS MAIS DE 1300 PARA TREINOS E MAIS DE 500 PARA TESTES.

O site da YOLO é https://pjreddie.com/darknet/yolo/https://pjreddie.com/darknet/yolo/

## Index 


Detection Using A Pre-Trained Model
YOLO significa You only look once

Executei esses comandos no terminal do Linux

git clone https://github.com/pjreddie/darknet
cd darknet
make

depois carreguei os pesos
wget https://pjreddie.com/media/files/yolov3.weights
Ficaram no subdiretorio da Darkenet cfg/

Depois executei a darknet e testei encontrar o cachorro na imagem, usando a yolov3
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg

O resultado acima é o encontrado pela YOLO
Resultado da detecção
dog: 99%
truck: 93%
bicycle: 99%

Testei a YOLO v3 usando a web cam, executando esse comando. Funciona bem
./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights

Toda a parte de VOC pulei

Fui para a Training YOLO on COCO (http://cocodataset.org/#overview)
COCO é um dataset de imagens marcadas a sigla significa Common Objects in Context

Para treinar a YOLO precisa das imagens e dos labels (marcacoes) , executando esses comandos é possível carregar as imagens e as marcacoes da COCO

cp scripts/get_coco_dataset.sh data
cd data
bash get_coco_dataset.sh

Depois que carregar a base da COCO tem que alterar um arquivo cfg/coco.data 
Criei um arquivo chamado coco_cps.data (cps é de cone, pare e siga)
Nao alterei os arquivos da COCO, na verdade criei outros para usar no treino depois

Esse é o que esta no arquivo coco_cps.data:

classes= 3
train  = /home/edufrigini/Desktop/darknet/data/coco/trainvalno5k_cps.txt
valid = /home/edufrigini/Desktop/darknet/data/coco/5k_cps.txt
eval = /home/edufrigini/Desktop/darknet/data/coco/5k_cps.txt
names = data/coco_cps.names
backup = backup

Sao 3 classes e para treino aponta para o arquivo trainvalno5k_cps.txt para onde tem a lista de nomes de todas a imagens para treino
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1826.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1827.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1828.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1829.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1830.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1831.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1832.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1833.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/treino_cps/IMG_1834.JPG
...

O arquivo 5k_cps.txt são todas as imagens para teste
/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_0001.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_0014.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_0169.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_0229.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_0308.JPG
/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_0334.JPG
...

O arquivo coco_cps.names são os nomes das tres classes
cone
pare
siga

Depois copiei o arquivo da yolov3.cfg e como yolov3_cps.cfg e alterei alguns parametros 
Coloquei o batch para 128 e subdivisoes = 16
Alterei esses parametros
[14:50, 12/7/2018] Eduardo.Frigini: [convolutional]
batch_normalize=1
filters=32
size=1
stride=1
pad=1
activation=leaky
[14:50, 12/7/2018] Eduardo.Frigini: [yolo]
mask = 6,7,8
anchors = 10,13,  16,30,  33,23,  30,61,  62,45,  59,119,  116,90,  156,198,  373,326
classes=80
num=9
jitter=.3
ignore_thresh = .7
truth_thresh = 1
random=1

Depois alterei os filters para de 255 para 24 conforme orientacao do Ranik

De acordo com essa conta que ele passou:
“Vc deve ter mudado o numero de classes para 3
mas ai vc tem q mudar o numero de filters para (NC+5)*3)”

Depois disso treinei a rede usando a GPU com as imagens de cones, placas de siga e de pare, usando esse comando
./darknet detector train cfg/coco_cps.data cfg/yolov3_cps.cfg darknet53.conv.74 -gpus 0

Após 5 dias de treino a rede havia feito 80 mil interacoes e usei para os testes

Usei o mesmo arquivo coco_cps.data
Fiz uma copia da yolov3_cps.cfg para yolov3_cps_teste.cfg e alterei o inicio do arquivo. Apenas coloquei o batch igual a 1 e as subdivisoes para 1 tambem. Conforme script abaixo.
[net]
# Testing
batch=1
subdivisions=1
# Training
# batch=128
# subdivisions=16
width=320
height=320
channels=3
momentum=0.9
decay=0.0005
angle=0
saturation = 1.5
exposure = 1.5
hue=.1

Para testar a rede 

./darknet detector test cfg/coco_cps.data cfg/yolov3_cps_teste.cfg yolov3_cps_80000.weights

Resultado do comando
...
   99 conv    128  1 x 1 / 1    40 x  40 x 384   ->    40 x  40 x 128  0.157 BFLOPs
  100 conv    256  3 x 3 / 1    40 x  40 x 128   ->    40 x  40 x 256  0.944 BFLOPs
  101 conv    128  1 x 1 / 1    40 x  40 x 256   ->    40 x  40 x 128  0.105 BFLOPs
  102 conv    256  3 x 3 / 1    40 x  40 x 128   ->    40 x  40 x 256  0.944 BFLOPs
  103 conv    128  1 x 1 / 1    40 x  40 x 256   ->    40 x  40 x 128  0.105 BFLOPs
  104 conv    256  3 x 3 / 1    40 x  40 x 128   ->    40 x  40 x 256  0.944 BFLOPs
  105 conv     24  1 x 1 / 1    40 x  40 x 256   ->    40 x  40 x  24  0.020 BFLOPs
  106 yolo
Loading weights from yolov3_cps_80000.weights...Done!
Enter Image Path: 

/home/edufrigini/Desktop/darknet/data/coco/images/teste_cps/IMG_1681.JPG

Resultado do teste feito na rede treinada com as imagens de SIGA e PARE







Para marcar as fotos foi feito o seguinte script em C

/*******************GENERATE_GT***********************

 Compile:
 g++ -std=c++0x -o generate_gt generate_gt_cone_stop_go.cpp -W -Wall `pkg-config --cflags opencv` -O4 `pkg-config --libs opencv` -Wno-unused-variable

 *************************************************/


#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include "opencv2/imgproc/imgproc_c.h"

#include <stdio.h>
#include <sys/io.h>
#include <string>
#include <vector>
#include <fstream>
#include <sstream>
#include <iostream>
#include <dirent.h>
#include <sys/types.h>


using namespace std;
using namespace cv;


//#define SPECIFIC_LABEL "trafficlight"         // Uncomment to see only images that have a specific annotation, like traffic_light
double RESIZE = 0.3;

typedef struct
{
	int x0;
	int y0;
	int x1;
	int y1;
	string state;
} BBOX;


Mat image, image2;
BBOX global_bbox;
vector<BBOX> bbox_vector, previous_bbox_vector;
string window_name = "<G>Go  <Y>Cone  <R>Stop  <N>Next  <B>Back  <P>Reuse_Previous_Mark  <ESC>Exit";


void
drawing_current_bbox(int x0, int y0, int x1, int y1)
{
	Mat img;

	if (image2.empty())
		img = image.clone();
	else
		img = image2.clone();

	rectangle(img, cvPoint(x0, y0), cvPoint(x1, y1), CV_RGB(0, 255, 255), 1);
	imshow(window_name, img);
	img.~Mat();
}


void
drawl_all_bbox()
{
	int r = 0, g = 0, b = 0;

	if (!image2.empty())
		image2.~Mat();

	image2 = image.clone();

	for (unsigned int i = 0; i < bbox_vector.size(); i++)
	{
		if (bbox_vector[i].state.compare("Stop_Sign") == 0)
		{
			r = 255; g = 0; b = 0;
		}
		else if (bbox_vector[i].state.compare("Traffic_Cone") == 0)
		{
			r = 200; g = 200; b = 0;
		}
		else if (bbox_vector[i].state.compare("Go_Sign") == 0)
		{
			r = 20; g = 150; b = 50;
		}
		else //if (bbox_vector[i].state.compare("OffTrafficLight") == 0)
		{
			r = 200; g = 200; b = 200;
		}
		rectangle(image2, cvPoint(bbox_vector[i].x0, bbox_vector[i].y0), cvPoint(bbox_vector[i].x1, bbox_vector[i].y1), CV_RGB(r, g, b), 1);
	}
	imshow(window_name, image2);
}


void
move_bbox(int x0_displacement, int y0_displacement, int x1_displacement, int y1_displacement)
{
	global_bbox.x0 += x0_displacement;
	global_bbox.y0 += y0_displacement;
	global_bbox.x1 += x1_displacement;
	global_bbox.y1 += y1_displacement;

	drawing_current_bbox(global_bbox.x0, global_bbox.y0, global_bbox.x1, global_bbox.y1);
}


void
resize_bbox(int zoom)
{
	global_bbox.x0 -= zoom;
	global_bbox.y0 -= zoom;
	global_bbox.x1 += zoom;
	global_bbox.y1 += zoom;

	drawing_current_bbox(global_bbox.x0, global_bbox.y0, global_bbox.x1, global_bbox.y1);
}


bool
click_is_inside_bbox(int x, int y)
{
	bool control = false;
	vector<BBOX> aux;

	//cout << bbox_vector.size() << endl;

	for (unsigned int i = 0; i < bbox_vector.size(); i++)
	{
		if ((x >= bbox_vector[i].x0) && (x <= bbox_vector[i].x1) && (y >= bbox_vector[i].y0) && (y <= bbox_vector[i].y1))
		{
			//cout << "entrou" << endl;
			global_bbox.x0 = bbox_vector[i].x0;
			global_bbox.y0 = bbox_vector[i].y0;
			global_bbox.x1 = bbox_vector[i].x1;
			global_bbox.y1 = bbox_vector[i].y1;
			control = true;
		}
		else
		{
			aux.push_back(bbox_vector[i]);
		}
	}
	bbox_vector.clear();
	bbox_vector = aux;
	aux.clear();

	return control;
}


void
on_mouse(int event, int x, int y, int, void*)
{
	static bool startDraw = false;

	if (event == EVENT_LBUTTONDOWN)
	{
		if (!startDraw)
		{
			if (click_is_inside_bbox(x, y))
			{
				drawl_all_bbox();
				drawing_current_bbox(global_bbox.x0, global_bbox.y0, global_bbox.x1, global_bbox.y1);
			}
			else
			{
				global_bbox.x0 = x;
				global_bbox.y0 = y;
				startDraw = true;
			}
		}
		else
		{
			global_bbox.x1 = x;
			global_bbox.y1 = y;
			startDraw = false;
		}
	}
	if (event == EVENT_MOUSEMOVE && startDraw)
	{
		drawing_current_bbox(global_bbox.x0, global_bbox.y0, x, y);
	}
}


void
could_not_open_error_message(string type, string name)
{
	cerr << "\n" <<
	"------------------------------------------------------------------------------------------" << endl <<
	"Failed! COLD NOT OPEN " << type << ": " << name << endl <<
	"------------------------------------------------------------------------------------------" << "\n\n";
}


vector<string>
open_file_read_all_image_names(string filename)
{
	ifstream input;
	string line;
	vector<string> image_name_vector;

	input.open(filename.c_str());

	if (input.is_open())
	{
		getline(input, line);
		while (!input.eof())
		{
			image_name_vector.push_back(line);
			getline(input, line);
		}
		input.close();

		return image_name_vector;
	}
	else
	{
		could_not_open_error_message("FILE", filename);
		exit(0);
	}
}


string
get_image_name(string image_path)
{
	string s;

	istringstream iss(image_path);
	for (unsigned int i = 0; getline(iss, s, '/'); )
	{
		i++;
	}

	return s;
}


void
sort_coordinates(int &x0, int &y0, int &x1, int &y1)
{
	int aux = 0;

	if (x0 > x1)
	{
		aux = x0;
		x0 = x1;
		x1 = aux;
	}

	if (y0 > y1)
	{
		aux = y0;
		y0 = y1;
		y1 = aux;
	}
}


int
open_image_label_file(string image_name)
{
	int label_found = 0; // FALSE
	ifstream current_label_file;
	string line, aux;
	BBOX bbox;
	vector<string> line_vector;

	image_name = get_image_name(image_name);
	image_name.replace(image_name.size()-3, 3, "txt");
	image_name = "labels/" + image_name;
	cout << "Loading label: " << image_name << endl << endl;

	current_label_file.open(image_name.c_str());

	if (!current_label_file.is_open())
		return label_found;

	getline(current_label_file, line);
	while (!current_label_file.eof())
	{
		istringstream iss(line);
		for (unsigned int i = 0; getline(iss, aux, ' '); i++)
			line_vector.push_back(aux);

		#ifdef SPECIFIC_LABEL
		if (line_vector[0].compare(SPECIFIC_LABEL) == 0)   // Compare returns the number off different characteres
			label_found = 1;
		#endif

		if (line_vector[0].compare("DontCare"))      // If this is not a DontCare line
		{
			bbox.x0 = stoi(line_vector[4]) * RESIZE;
			bbox.y0 = stoi(line_vector[5]) * RESIZE;
			bbox.x1 = stoi(line_vector[6]) * RESIZE;
			bbox.y1 = stoi(line_vector[7]) * RESIZE;
			bbox.state = line_vector[0];

			sort_coordinates(bbox.x0, bbox.y0, bbox.x1, bbox.y1);

			bbox_vector.push_back(bbox);
		}
		line_vector.clear();
		getline(current_label_file, line);
	}

	return label_found;
}

void
consolidate_marked_bbox(string state)
{
	int aux;

	global_bbox.state = state;

	if (global_bbox.x0 == global_bbox.x1 || global_bbox.y0 == global_bbox.y1)
				return;

	if (global_bbox.x0 > global_bbox.x1 || global_bbox.y0 > global_bbox.y1)
	{
		aux = global_bbox.x0;
		global_bbox.x0 = global_bbox.x1;
		global_bbox.x1 = aux;

		aux = global_bbox.y0;
		global_bbox.y0 = global_bbox.y1;
		global_bbox.y1 = aux;
	}

	bbox_vector.push_back(global_bbox);
	drawl_all_bbox();
}


void
set_previous_mark_to_image()
{
	bbox_vector = previous_bbox_vector;
	drawl_all_bbox();
}


void
save_to_file(string image_name)
{
	ofstream current_label_file;

	image_name = get_image_name(image_name);

	image_name.replace(image_name.size()-3, 3, "txt");
	image_name = "labels/" + image_name;

	current_label_file.open(image_name.c_str());

	cout << "Saving label file: " << image_name << endl;

	for (unsigned int i = 0; i < bbox_vector.size(); i++)
	{
		sort_coordinates(bbox_vector[i].x0, bbox_vector[i].y0, bbox_vector[i].x1, bbox_vector[i].y1);

		current_label_file << bbox_vector[i].state + ' ' + "0.00" + ' ' + "0" + ' ' + "0.00" + ' ' + to_string(ceil(bbox_vector[i].x0/RESIZE)) + ".00" + ' ' + to_string(ceil(bbox_vector[i].y0/RESIZE)) + ".00" + ' ' + to_string(ceil(bbox_vector[i].x1/RESIZE)) +
				".00" + ' '	+ to_string(ceil(bbox_vector[i].y1/RESIZE)) + ".00" + ' ' + "0.00" + ' ' + "0.00" + ' ' + "0.00" + ' ' + "0.00" + ' ' + "0.00" + ' ' + "0.00" + ' ' + "0" + "\n";
	}

	previous_bbox_vector = bbox_vector;
	bbox_vector.clear();
	current_label_file.close();
}


string
get_path(string image_list)
{
	string s, path;
	vector<string> str_vector;

	istringstream iss(image_list);
	for (unsigned int i = 0; getline(iss, s, '/'); i++)
	{
		str_vector.push_back(s);
	}
	str_vector.pop_back();

	for (unsigned int i = 0; i < str_vector.size(); i++)
	{
		path += str_vector[i] + '/';
	}
	str_vector.clear();
	return path;
}


void
apply_zoom()
{
	static bool zoom = false;
	static int x0, y0, x1, y1;
	static double resize_factor_x, resize_factor_y;

	if (zoom)
	{
		global_bbox.x0 = (global_bbox.x0 * resize_factor_x) + x0;
		global_bbox.y0 = (global_bbox.y0 * resize_factor_y) + y0;
		global_bbox.x1 = (global_bbox.x1 * resize_factor_x) + x0;
		global_bbox.y1 = (global_bbox.y1 * resize_factor_y) + y0;

		image = image.clone();
		image2.~Mat();

		drawl_all_bbox();
		drawing_current_bbox(global_bbox.x0, global_bbox.y0, global_bbox.x1, global_bbox.y1);

		zoom = false;
	}
	else
	{
		if (global_bbox.x0 > global_bbox.x1)
		{
			int aux = global_bbox.x0;
			global_bbox.x0 = global_bbox.x1;
			global_bbox.x1 = aux;
		}
		if (global_bbox.y0 > global_bbox.y1)
		{
			int aux = global_bbox.y0;
			global_bbox.y0 = global_bbox.y1;
			global_bbox.y1 = aux;
		}

		if (global_bbox.x0 - 30 > 0)
			x0 = global_bbox.x0 - 30;
		else
			x0 = 0;
		if (global_bbox.y0 - 30 > 0)
			y0 = global_bbox.y0 - 30;
		else
			y0 = 0;
		if (global_bbox.x1 + 30 < image.cols)
			x1 = global_bbox.x1 + 30;
		else
			x1 = image.cols;
		if (global_bbox.y1 + 30 < image.rows)
			y1 = global_bbox.y1 + 30;
		else
			y1 = image.rows;

		resize_factor_x = (double)(x1 - x0) / image2.cols;
		resize_factor_y = (double)(y1 - y0) / image2.rows;

		Rect myROI(x0, y0, x1 - x0, y1 - y0);

		image2 = image(myROI);
		resize(image2, image2, Size(image.cols, image.rows));
		imshow(window_name, image2);

		zoom = true;
	}
}


void
check_buton_pressed(int iKey, vector<string> image_name_vector, unsigned int &actual_image_position)
{
	static unsigned int move_edge = 0;

	switch (iKey)
	{
		case 114:                      // R key 114 Save added rectangles to RED file
			consolidate_marked_bbox("Stop_Sign");
			break;

		case 121:                      // Y  key 121 Save added rectangles to YELOW file
			consolidate_marked_bbox("Traffic_Cone");
			break;

		case 103:                     // G key 103 Save added rectangles to GREEN file
			consolidate_marked_bbox("Go_Sign");
			break;

		case 111:                       // O key 111 Save added rectangles to OFF file
			consolidate_marked_bbox("OffTrafficLight");
			break;

		case 112:                       // <P> key 112 set the previous mark to the current image, it does not work if the image is already marked
			set_previous_mark_to_image();
			break;

		case 110:                        // N key 110 Go to next image in the list
			save_to_file(image_name_vector[actual_image_position]);
			actual_image_position++;
			break;

		case 98:                        // B key 98 Go back one image in the list
			save_to_file(image_name_vector[actual_image_position]);
			if (actual_image_position > 0)
				actual_image_position--;
			break;

		case 49:                      // 49 <1> Select left vertical bbox edge
			move_edge = 1;
			break;

		case 50:                      // 50 <2> Select top horizontal bbox edge
			move_edge = 2;
			break;

		case 51:                      // 51 <3> Select right vertical bbox edge
			move_edge = 3;
			break;

		case 52:                      // 52 <4> Select bottom horizontal bbox edge
			move_edge = 4;
			break;

		case 48:                      // 48 <0> Go back to move the hole bbox
			move_edge = 0;
			break;

		case 119:                     // 65362 <W> Up
			if (move_edge == 0)
				move_bbox(0, -1, 0, -1);
			else if (move_edge == 2)
				move_bbox(0, -1, 0, 0);
			else if (move_edge == 4)
				move_bbox(0, 0, 0, -1);
			break;

		case 115:                     // 65364 <S> Down
			if (move_edge == 0)
				move_bbox(0, 1, 0, 1);
			else if (move_edge == 2)
				move_bbox(0, 1, 0, 0);
			else if (move_edge == 4)
				move_bbox(0, 0, 0, 1);
			break;

		case 97:                     // 65361 <L> Left
			if (move_edge == 0)
				move_bbox(-1, 0, -1, 0);
			else if (move_edge == 1)
				move_bbox(-1, 0, 0, 0);
			else if (move_edge == 3)
				move_bbox(0, 0, -1, 0);
			break;

		case 100:                     // 65363 <R> Right
			if (move_edge == 0)
				move_bbox(1, 0, 1, 0);
			else if (move_edge == 1)
				move_bbox(1, 0, 0, 0);
			else if (move_edge == 3)
				move_bbox(0, 0, 1, 0);
			break;

		case 113:                     // 113 <Q> Enlarge
			resize_bbox(1);
			break;

		case 101:                     // 101 <E> Reduce
			resize_bbox(-1);
			break;

		case 122:                     // 122 <Z> Reduce
			apply_zoom();
			break;

		case 27:                        // ESC -> key 27 Exit program
			cout << "\nPROGRAM EXITED BY PRESSING <ESC> KEY!" << "\n\n";
			break;
	}
}


int
main(int argc, char** argv)
{
	int iKey = -1;
	unsigned int actual_image_position = 0;
	vector<string> image_name_vector;


	if (argc != 2)
	{
		cerr << argv[0] << " <input_ImageList.txt>" << endl;
		return -1;
	}

	//path = get_path(argv[1]);
	image_name_vector = open_file_read_all_image_names (argv[1]);

	namedWindow(window_name, WINDOW_AUTOSIZE);
	setMouseCallback(window_name, on_mouse);

	while (actual_image_position < image_name_vector.size() && iKey != 27)
	{
		cout << "Loading image: " << image_name_vector[actual_image_position] << endl;
		image = imread(image_name_vector[actual_image_position], 1);

		if (!image.empty())
		{
			resize(image, image, Size(image.cols * RESIZE, image.rows * RESIZE));

			imshow(window_name, image);
			int label_found = open_image_label_file(image_name_vector[actual_image_position]);

			// Uncomment SPECIFIC_LABEL to see only images that have a specific annotation, like traffic_light
			#ifdef SPECIFIC_LABEL
			if (!label_found)
			{
				previous_bbox_vector = bbox_vector;
				bbox_vector.clear();
				actual_image_position++;
				continue;
			}
			#endif

			drawl_all_bbox();

			iKey = -1;												// 98  <B> go back one image in the list
			while (iKey != 98 && iKey != 110 && iKey != 27) 		// 110 <N> go to next image saving nothing in no file
			{														// 27  <ESC> go to next image saving nothing in no file
				iKey = waitKey(0) & 0xff;
				//cout << "IKEY " << iKey << endl;
				check_buton_pressed(iKey, image_name_vector, actual_image_position);
			}
		}
		else
		{
			could_not_open_error_message("IMAGE", image_name_vector[actual_image_position]);
		}
	}
	bbox_vector.clear();
	previous_bbox_vector.clear();
	image_name_vector.clear();
	image.~Mat();
	image2.~Mat();
	destroyWindow(window_name);

	return EXIT_SUCCESS;
}
