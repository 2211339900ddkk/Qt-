# Qt-
开发记录
/*********************************************************************
*
* ANSI C Example program:
*    Acq-IntClk.c
*
* Example Category:
*    AI
*
* Description:
*    This example demonstrates how to acquire a finite amount of data
*    using the DAQ device's internal clock.
*
* Instructions for Running:
*    1. Select the physical channel to correspond to where your
*       signal is input on the DAQ device.
*    2. Enter the minimum and maximum voltages.
*    Note: For better accuracy try to match the input range to the
*          expected voltage level of the measured signal.
*    3. Select the number of samples to acquire.
*    4. Set the rate of the acquisition.
*    Note: The rate should be AT LEAST twice as fast as the maximum
*          frequency component of the signal being acquired.
*
* Steps:
*    1. Create a task.
*    2. Create an analog input voltage channel.
*    3. Set the rate for the sample clock. Additionally, define the
*       sample mode to be finite and set the number of samples to be
*       acquired per channel.
*    4. Call the Start function to start the acquisition.
*    5. Read all of the waveform data.
*    6. Call the Clear Task function to clear the task.
*    7. Display an error if any.
*
* I/O Connections Overview:
*    Make sure your signal input terminal matches the Physical
*    Channel I/O Control. For further connection information, refer
*    to your hardware reference manual.
*
*********************************************************************/

#include <stdio.h>
#include "Art_DAQ.h" 
#include <malloc.h>
#include <string.h>
#include <stdlib.h>
#include <conio.h>
#include <iostream>


#define ArtDAQErrChk(functionCall) if( ArtDAQFailed(error=(functionCall)) ) goto Error; else
#pragma warning(disable : 4996)


int main(int argc, char* argv[])
{
if (((int)argv[0] * (int)argv[1] * (int)argv[2]) == 0)
{
	printf("zixi");
	std::cout << argc << std::endl;//输出参数的个数
	for (int i = 0; i < argc+1; i++) {
	std::cout << argv[i] << std::endl;//输出具体参数
	}
	if (*argv[1] == '1') {//提取第一个参数，
		std::cout << "zixi" << std::endl;
	}

	getchar();
	return();

}
else
{
	printf("进入程序");
	std::cout << argc << std::endl;//输出参数的个数
	for (int i = 0; i < argc; i++) {
		std::cout << argv[i] << std::endl;//输出具体参数
	}
	if (*argv[1] == '1') {//提取第一个参数，
		std::cout << "hello codelab!" << std::endl;
	}

	int32       error = 0;
	TaskHandle  taskHandle = 0;
	int32       read;
	char        errBuff[2048] = { '\0' };


	float64* data;
	data = (float64*)malloc((int)argv[1] * (int)argv[2] * sizeof(float64));

	int64  i;
	int64  j;

	/*********************************************/
	// ArtDAQ Configure Code
	/*********************************************/
	ArtDAQErrChk(ArtDAQ_CreateTask("", &taskHandle));
	ArtDAQErrChk(ArtDAQ_CreateAIVoltageChan(taskHandle, "Dev1/ai0", "Dev1/ai1", ArtDAQ_Val_Cfg_Default, -1.0, 1.0, ArtDAQ_Val_Volts, NULL));
	ArtDAQErrChk(ArtDAQ_CfgSampClkTiming(taskHandle, "", 100000.0, ArtDAQ_Val_Rising, ArtDAQ_Val_FiniteSamps, (int)argv[1] * (int)argv[2]));

	/*********************************************/
	// ArtDAQ Start Code
	/*********************************************/
	ArtDAQErrChk(ArtDAQ_StartTask(taskHandle));

	/*********************************************/
	// ArtDAQ Read Code
	/*********************************************/
	ArtDAQErrChk(ArtDAQ_ReadAnalogF64(taskHandle, 250000, 10.0, ArtDAQ_Val_GroupByChannel, data, (int)argv[1] * (int)argv[2], &read, NULL));

	printf("Acquired %d samples\n", (int)read);


	FILE* fp;					// 声明一个文件流类型的变量，FILE 为 stdio.h 里定义的

	fp = fopen("C:/Users/klbb_/Desktop/value.txt", "w+");	 //用 fopen 函数打开文件，第一个参数表示文件名，若不是当前
															// 路径请加 \\ 号，如 C:\\Windows\\System32，"w" 表示写(write)

	setbuf(fp, NULL);


	/*********************************************/
	// 申请二维动态内存
	/*********************************************/

	{
		float64(*p)[500] = (float64(*)[500])malloc(sizeof(float64) * 500 * 500);


		/*********************************************/
		// 将一维数组转变成二维数组
		/*********************************************/

		for (i = 0; i < (int)argv[1]; i++)
		{
			for (j = 0; j < (int)argv[2]; j++)
			{
				p[i][j] = data[i * (int)argv[1] + j];
			}
		}
		/*********************************************/
		// 打印出二维数组并生成TXT文件
		/*********************************************/


		for (j = 0; j < (int)argv[1]; j++)
		{

			for (i = 0; i < (int)argv[2]; i++)
			{
				printf("%lf\t", p[i][j]);
				fprintf(fp, "%lf\t", p[i][j]);
				fflush(fp);  //清除缓存
			}
			printf("\n");

		}
	}






Error:
	if (ArtDAQFailed(error))
		ArtDAQ_GetExtendedErrorInfo(errBuff, 2048);
	if (taskHandle != 0) {
		/*********************************************/
		// ArtDAQ Stop Code
		/*********************************************/
		ArtDAQ_StopTask(taskHandle);
		ArtDAQ_ClearTask(taskHandle);
	}
	if (ArtDAQFailed(error))
		printf("ArtDAQ_ Error: %s\n", errBuff);
	printf("Acquired %d samples\n", (int)read);
	printf("End of program, press Enter key to quit\n");
	//free(data);
	getchar();
}
	return 0;
}
