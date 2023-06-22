#include <iostream>
#include <math.h>
#include <stdlib.h>
#include<string.h>
#include<msclr\marshal_cppstd.h>
#include<mpi.h>
#include<stdio.h>
#include <ctime>
#pragma once
#using <mscorlib.dll>
#using <System.dll>
#using <System.Drawing.dll>
#using <System.Windows.Forms.dll>

using namespace std;
using namespace msclr::interop;


// takes the width and hight and the path of the image  
int* inputImage(int* w, int* h, System::String^ imagePath)
{
	int* input;
	int OriginalImageWidth, OriginalImageHeight;

	//Reading Image 
	System::Drawing::Bitmap BM(imagePath);

	OriginalImageWidth = BM.Width;
	OriginalImageHeight = BM.Height;
	*w = BM.Width;
	*h = BM.Height;
	int* Red = new int[BM.Height * BM.Width];
	int* Green = new int[BM.Height * BM.Width];
	int* Blue = new int[BM.Height * BM.Width];
	input = new int[BM.Height * BM.Width];
	for (int i = 0; i < BM.Height; i++)
	{
		for (int j = 0; j < BM.Width; j++)
		{
			System::Drawing::Color c = BM.GetPixel(j, i);  //Gets the color of the specified pixels in this Image Bitmap

			//getting the RGB colors
			Red[i * BM.Width + j] = c.R;
			Blue[i * BM.Width + j] = c.B;
			Green[i * BM.Width + j] = c.G;

			//getting the grey scale by getting the average of the RGB values
			input[i * BM.Width + j] = ((c.R + c.B + c.G) / 3); 
			
		}

	}
	return input;
}


void createImage(int* image, int width, int height, int index)
{
	System::Drawing::Bitmap NewImage(width, height);


	for (int i = 0; i < NewImage.Height; i++)
	{
		for (int j = 0; j < NewImage.Width; j++)
		{
			//make sure 8 bit depth are used per color component between black and white

			if (image[i * width + j] < 0)
			{
				image[i * width + j] = 0;
			}
			if (image[i * width + j] > 255)
			{
				image[i * width + j] = 255;
			}

			//creating the color strucure for each pixel 
			System::Drawing::Color c = System::Drawing::Color::FromArgb(image[i * NewImage.Width + j], image[i * NewImage.Width + j], image[i * NewImage.Width + j]);
			NewImage.SetPixel(j, i, c);
		}
	}
	NewImage.Save("..//Data//Output//output saved " + index + ".png");
	cout << "output Image " << index << " saved" << endl;
}


int main()
{
	int ImageWidth = 4, ImageHeight = 4;

	int start_time, stop_time, TotalTime = 0;

	System::String^ imagePath;
	std::string img;
	img = "..//Data//Input//test.png";

	imagePath = marshal_as<System::String^>(img);
	int* greyimage = inputImage(&ImageWidth, &ImageHeight, imagePath);

	MPI_Init(NULL, NULL);
	int rank, size;
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	MPI_Comm_size(MPI_COMM_WORLD, &size);

	int totalImgSize = ImageWidth * ImageHeight;
	int kernal[9] = { 0,-1,0,-1,4,-1,0,-1,0 };   //kernal of the high pass filter

	MPI_Bcast(&kernal, 9, MPI_INT, 0, MPI_COMM_WORLD); // sending the kerenl data to all processes

	int* PartialImage = new int[totalImgSize / size];
	int blocksOfPixels = (totalImgSize / size);
	int* PartialFilteredImage = new int[totalImgSize / size];

	//-------data buffer,count of data,datatype,recieve buffer,recieve count ,recievetype,from source process(0)
	MPI_Scatter(greyimage, blocksOfPixels, MPI_INT, PartialImage, blocksOfPixels, MPI_INT, 0, MPI_COMM_WORLD);
	int index;
	int sum = 0;
	int MovedWidth;
	for (int i = 0; i < blocksOfPixels; i++) 
	{
		sum = 0;
		index = i;
		MovedWidth = 1;
		for (int j = 0; j < 9; j++)
		{
			sum += (PartialImage[index] * kernal[j]); //sum the products of total kernel with ith block in partialImage

			if (MovedWidth % 3 == 0) 
			{
				index += (ImageWidth - index);
			}
			else
			{
				index++;
			}
			MovedWidth++;
		}
		PartialFilteredImage[i] = sum;
	}

	int* Final_Image = new int[totalImgSize];
	   //data buffer, count of data, datatype, recieve buffer, recieve count, recievetype, to source process(0)
	MPI_Gather(PartialFilteredImage, blocksOfPixels, MPI_INT, Final_Image, blocksOfPixels, MPI_INT, 0, MPI_COMM_WORLD);

	if (rank == 0) 
	{

		start_time = clock();
		createImage(greyimage, ImageWidth, ImageHeight, 0);
		stop_time = clock();
		TotalTime += (stop_time - start_time) / double(CLOCKS_PER_SEC) * 1000;
		cout << "time taken : " << TotalTime << endl;

		start_time = clock();
		createImage(Final_Image, ImageWidth, ImageHeight, 1);
		stop_time = clock();
		TotalTime += (stop_time - start_time) / double(CLOCKS_PER_SEC) * 1000;
		cout << "time taken: " << TotalTime << endl;
	}
	MPI_Finalize();
	free(greyimage);
	return 0;

}
