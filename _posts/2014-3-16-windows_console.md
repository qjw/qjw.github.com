---
layout: post
title: Windows直接写cmd绘制复杂图形
category: windows
---

## 源码

	#include <Windows.h>
	#include <tchar.h>
	#include <string>
	#include <time.h>
	#include <cstdint>
	#include <cassert>

	class Console
	{
	public:
		Console() :
			hStdOutput(INVALID_HANDLE_VALUE)
		{
		}
		bool Open(void)
		{
			hStdOutput = GetStdHandle(STD_OUTPUT_HANDLE);
			return INVALID_HANDLE_VALUE != hStdOutput;
		}
		inline bool SetTitle(const TCHAR* title) // 设置标题
		{
			return TRUE == SetConsoleTitle(title);
		}
		bool RemoveCursor(void) // 去处光标
		{
			CONSOLE_CURSOR_INFO cci;
			if (!GetConsoleCursorInfo(hStdOutput, &cci)) return false;
			cci.bVisible = false;
			if (!SetConsoleCursorInfo(hStdOutput, &cci)) return false;
			cci.bVisible = false;
			return true;
		}
		bool SetWindowRect(short x, short y) // 设置窗体尺寸
		{
			SMALL_RECT wrt = { 0, 0, x, y };
			if (!SetConsoleWindowInfo(hStdOutput, TRUE, &wrt)) return false;
			return true;
		}
		bool SetBufSize(short x, short y) // 设置缓冲尺寸
		{
			COORD coord = { x, y };
			if (!SetConsoleScreenBufferSize(hStdOutput, coord)) return false;
			return true;
		}

		bool GotoXY(short x, short y) // 移动光标
		{
			COORD coord = { x + x, y };
			if (!SetConsoleCursorPosition(hStdOutput, coord)) return false;
			return true;
		}
		bool SetColor(WORD color) // 设置前景色/背景色
		{
			if (!SetConsoleTextAttribute(hStdOutput, color)) return false;
			//if (!SetConsoleTextAttribute(hStdError, color)) return false;
			return true;
		}
		bool OutputString(const TCHAR* pstr, size_t len = 0) // 输出字符串
		{
			DWORD n = 0;
			return TRUE == WriteConsole(hStdOutput, pstr, len ? len : _tcslen(pstr), &n, NULL);
		}

		bool OutputStringNoMove(short x, short y, const TCHAR* pstr, size_t len = 0) // 输出字符串
		{
			COORD coord = { x + x, y };
			DWORD n = 0;
			if (TRUE != WriteConsoleOutputCharacter(hStdOutput, pstr, len ? len : _tcslen(pstr), coord, &n))
				return false;
			if (len == 1 && *pstr == _T('-'))
			{
				COORD coord1 = { x + x + 1, y };
				n = 0;
				if (TRUE != WriteConsoleOutputCharacter(hStdOutput, pstr, len ? len : _tcslen(pstr), coord1, &n))
					return false;
			}
			return true;
		}
	private:
		HANDLE hStdOutput;
	};

	int _tmain(int argc, _TCHAR* argv[])
	{
		srand(time(NULL));
		system("color F0");

		Console console_;
		if (!console_.Open())
			return -1;
		console_.SetTitle(_T("小游戏"));
		console_.RemoveCursor();
		console_.SetWindowRect(55 - 1, 40 - 1);
		console_.SetBufSize(55, 40);
		const short cheer_size_ = 5;
		const short cheer_width_ = 6;

		// 横着
		const short total_ = (cheer_size_ - 1)*cheer_width_;
		for (short i = 0; i < cheer_size_; i++)
		{
			for (short j = 0; j < total_; j++)
			{
				console_.OutputStringNoMove(j, i*cheer_width_, _T("-"), 1);
			}
		}
		// 竖直
		for (short i = 0; i < cheer_size_; i++)
		{
			for (short j = 0; j < total_; j++)
			{
				console_.OutputStringNoMove(i*cheer_width_, j, _T("|"), 1);
			}
		}
		// 左上 -> 右下
		for (short i = 0; i < cheer_size_ - 1; i += 2)
		{
			for (short j = 0; j < total_ - i*cheer_width_; j++)
			{
				console_.OutputStringNoMove(j + i*cheer_width_, j, _T("\\"), 1);
			}
		}
		for (short i = 2; i < cheer_size_ - 1; i += 2)
		{
			for (short j = 0; j < total_ - i*cheer_width_; j++)
			{
				console_.OutputStringNoMove(j, j + i*cheer_width_, _T("\\"), 1);
			}
		}
		// 左下 -> 右上
		for (short i = 0; i < cheer_size_ - 1; i += 2)
		{
			for (short j = 0; j < total_ - i*cheer_width_; j++)
			{
				console_.OutputStringNoMove(total_ - j - i*cheer_width_, j, _T("/"), 1);
			}
		}
		for (short i = 0; i < cheer_size_ - 1; i += 2)
		{
			for (short j = 0; j < total_ - i*cheer_width_; j++)
			{
				console_.OutputStringNoMove(total_ - j, j + i*cheer_width_, _T("/"), 1);
			}
		}

		const WORD red_ = FOREGROUND_RED | FOREGROUND_INTENSITY;
		const WORD green_ = FOREGROUND_GREEN | FOREGROUND_INTENSITY;
		const WORD plain_ = FOREGROUND_RED |
			FOREGROUND_GREEN |
			BACKGROUND_RED |
			BACKGROUND_GREEN |
			BACKGROUND_BLUE |
			BACKGROUND_INTENSITY;
		uint32_t cnt_ = 0;
		TCHAR buf_[36];
		buf_[2] = _T('\0');
		for (short i = 0; i < cheer_size_; i++)
		{
			for (short j = 0; j < cheer_size_; j++)
			{
				_sntprintf(buf_, _countof(buf_), _T("%02d"), cnt_);

				if (cnt_ < cheer_size_)
					console_.SetColor(red_);
				else if (cnt_ >= cheer_size_*(cheer_size_ - 1))
					console_.SetColor(green_);
				else
					console_.SetColor(plain_);

				console_.GotoXY(j*cheer_width_, i*cheer_width_);
				console_.OutputString(buf_, 2);
				cnt_++;
			}
		}

		while (true)
			Sleep(1000);
		return 0;
	}

##结果
	00----------01----------02----------03----------04
	| \         |         / | \         |         / |
	|   \       |       /   |   \       |       /   |
	|     \     |     /     |     \     |     /     |
	|       \   |   /       |       \   |   /       |
	|         \ | /         |         \ | /         |
	05----------06----------07----------08----------09
	|         / | \         |         / | \         |
	|       /   |   \       |       /   |   \       |
	|     /     |     \     |     /     |     \     |
	|   /       |       \   |   /       |       \   |
	| /         |         \ | /         |         \ |
	10----------11----------12----------13----------14
	| \         |         / | \         |         / |
	|   \       |       /   |   \       |       /   |
	|     \     |     /     |     \     |     /     |
	|       \   |   /       |       \   |   /       |
	|         \ | /         |         \ | /         |
	15----------16----------17----------18----------19
	|         / | \         |         / | \         |
	|       /   |   \       |       /   |   \       |
	|     /     |     \     |     /     |     \     |
	|   /       |       \   |   /       |       \   |
	| /         |         \ | /         |         \ |
	20----------21----------22----------23----------24
	
##参考
1. <http://blog.csdn.net/undefined_behavior/article/details/2067280>