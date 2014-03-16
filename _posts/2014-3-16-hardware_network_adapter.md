---
layout: post
title: Windows获取物理网卡外设
category: windows
---

##原理
使用**CM_Get_DevNode_Registry_Property_Ex**系列api获取所有的网络硬件设备，使用**SetupDiGetClassDevs**系列api获取已经启用的网络设备。从【**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Network\{4D36E972-E325-11CE-BFC1-08002BE10318}\{9082CA70-B700-4161-918F-33DCF436B119}\Connection**】读取所有的网卡。

##源码

	#include <Windows.h>
	#include <tchar.h>
	#include <string>
	#include <cassert>

	#if defined _UNICODE || defined UNICODE
	typedef std::wstring String;
	#else
	typedef std::string String;
	#endif

	#include <Cfgmgr32.h>
	#pragma comment(lib,"Cfgmgr32.lib")

	#include <setupapi.h>
	#pragma comment (lib,"Setupapi.lib")
	#define MACADDRESS_BYTELEN      6    
	#define OID_802_3_PERMANENT_ADDRESS 0x01010101
	#define IOCTL_NDIS_QUERY_GLOBAL_STATS 0x00170002
	GUID GUID_DEVCLASS_QUERY = { 0xAD498944, 0x762F, 0x11D0, 0x8D, 0xCB, 0x00, 0xC0, 0x4F, 0xC3, 0x35, 0x8C };



	HMACHINE m_hMachine;

	String GetHARDWAREID(DEVNODE DevNode, ULONG flag)
	{
		String	strType;
		String	strValue;
		String DisplayName;
		LPTSTR	Buffer = NULL;

		int  BufferSize = MAX_PATH + MAX_DEVICE_ID_LEN;
		ULONG  BufferLen = BufferSize * sizeof(TCHAR);

		Buffer = new TCHAR[BufferSize];
		if (CR_SUCCESS == CM_Get_DevNode_Registry_Property_Ex(DevNode,
			flag, NULL,
			Buffer, &BufferLen, 0, m_hMachine))
		{
			DisplayName = Buffer;
		}
		else
		{
			DisplayName = _T("error");
		}

		delete[] Buffer;
		return DisplayName;
	}

	String GetDeviceName(DEVNODE DevNode)
	{
		String	strType;
		String	strValue;
		String DisplayName;
		LPTSTR	Buffer = NULL;

		int  BufferSize = MAX_PATH + MAX_DEVICE_ID_LEN;
		ULONG  BufferLen = BufferSize * sizeof(TCHAR);

		Buffer = new TCHAR[BufferSize];
		if (CR_SUCCESS == CM_Get_DevNode_Registry_Property_Ex(DevNode,
			CM_DRP_FRIENDLYNAME, NULL,
			Buffer, &BufferLen, 0, m_hMachine))
		{
			DisplayName = Buffer;
		}
		else
		{
			BufferLen = BufferSize * sizeof(TCHAR);

			if (CR_SUCCESS == CM_Get_DevNode_Registry_Property_Ex(DevNode,
				CM_DRP_DEVICEDESC, NULL,
				Buffer, &BufferLen, 0, m_hMachine))
			{
				DisplayName = Buffer;
			}
			else
			{
				DisplayName = _T("Unknown Device");
			}
		}

		delete[] Buffer;
		return DisplayName;
	}

	void RetrieveSubNodes(DEVINST parent, DEVINST sibling, DEVNODE dn)
	{
		DEVNODE dnSibling, dnChild;

		do
		{
			CONFIGRET cr = CM_Get_Sibling_Ex(&dnSibling, dn, 0, m_hMachine);

			if (cr != CR_SUCCESS)
				dnSibling = NULL;

			TCHAR GuidString[MAX_GUID_STRING_LEN];
			ULONG Size = sizeof(GuidString);

			cr = CM_Get_DevNode_Registry_Property_Ex(dn, CM_DRP_CLASSGUID, NULL,
				GuidString, &Size, 0, m_hMachine);

			if (cr == CR_SUCCESS)
			{

				String DeviceName = GetHARDWAREID(dn, CM_DRP_CLASS);
				if (DeviceName == _T("Net"))
				{
					_tprintf(_T("CM_DRP_CLASS [%s]\n"), DeviceName.c_str());

					DeviceName = GetDeviceName(dn);
					_tprintf(_T("DeviceName [%s]\n"), DeviceName.c_str());

					DeviceName = GetHARDWAREID(dn, CM_DRP_HARDWAREID);
					_tprintf(_T("CM_DRP_HARDWAREID [%s]\n"), DeviceName.c_str());

					DeviceName = GetHARDWAREID(dn, CM_DRP_SERVICE);
					_tprintf(_T("CM_DRP_SERVICE [%s]\n"), DeviceName.c_str());


					DeviceName = GetHARDWAREID(dn, CM_DRP_CLASSGUID);
					_tprintf(_T("CM_DRP_CLASSGUID [%s]\n"), DeviceName.c_str());
				}

	#if 0
				GUID Guid;
				GuidString[MAX_GUID_STRING_LEN - 2] = _T('\0');
				UuidFromString(&GuidString[1], &Guid);
				int Index;
				if (SetupDiGetClassImageIndex(&m_ImageListData, &Guid, &Index))
				{
					String DeviceName = GetDeviceName(dn);
					m_Devices.InsertItem(m_Devices.GetItemCount(), DeviceName, Index);
				}
	#endif
			}

			cr = CM_Get_Child_Ex(&dnChild, dn, 0, m_hMachine);
			if (cr == CR_SUCCESS)
			{
				RetrieveSubNodes(dn, NULL, dnChild);
			}

			dn = dnSibling;

		} while (dn != NULL);

	}

	static BOOL GetMacAddress()
	{
		HDEVINFO hDevInfo;
		DWORD MemberIndex, RequiredSize;
		SP_DEVICE_INTERFACE_DATA            DeviceInterfaceData;
		PSP_DEVICE_INTERFACE_DETAIL_DATA    DeviceInterfaceDetailData;
		BOOL isOK = FALSE;

		// 获取设备信息集
		hDevInfo = SetupDiGetClassDevs(&GUID_DEVCLASS_QUERY, NULL, NULL, DIGCF_PRESENT | DIGCF_INTERFACEDEVICE);
		if (INVALID_HANDLE_VALUE == hDevInfo)
		{
			return FALSE;
		}

		//  枚举设备信息集中所有设备
		DeviceInterfaceData.cbSize = sizeof(SP_DEVICE_INTERFACE_DATA);

		MemberIndex = 0;
		while (SetupDiEnumDeviceInterfaces(hDevInfo, NULL, &GUID_DEVCLASS_QUERY, MemberIndex, &DeviceInterfaceData))
		{
			MemberIndex++;
			// 获取接收缓冲区大小，函数返回值为FALSE，GetLastError()=ERROR_INSUFFICIENT_BUFFER  
			SetupDiGetDeviceInterfaceDetail(hDevInfo, &DeviceInterfaceData, NULL, 0, &RequiredSize, NULL);

			// 申请接收缓冲区  
			DeviceInterfaceDetailData = (PSP_DEVICE_INTERFACE_DETAIL_DATA)malloc(RequiredSize);
			DeviceInterfaceDetailData->cbSize = sizeof(SP_DEVICE_INTERFACE_DETAIL_DATA);

			// 获取设备细节信息  
			if (SetupDiGetDeviceInterfaceDetail(hDevInfo, &DeviceInterfaceData,
				DeviceInterfaceDetailData, RequiredSize, NULL, NULL))
			{
				HANDLE hDeviceFile;

				_tprintf(_T("PATH [%s]\n"), 
					DeviceInterfaceDetailData->DevicePath);
				
				// 获取设备句柄  
				hDeviceFile = CreateFile(
					DeviceInterfaceDetailData->DevicePath,
					0,
					FILE_SHARE_READ | FILE_SHARE_WRITE,
					NULL,
					OPEN_EXISTING,
					0,
					NULL);

				if (hDeviceFile != INVALID_HANDLE_VALUE)
				{
					ULONG   dwID;
					BYTE    ucData[MACADDRESS_BYTELEN];
					DWORD   dwByteRet;

					// 获取原生MAC地址  
					dwID = OID_802_3_PERMANENT_ADDRESS;
					isOK = DeviceIoControl(hDeviceFile, IOCTL_NDIS_QUERY_GLOBAL_STATS, &dwID,
						sizeof(dwID), ucData, sizeof(ucData), &dwByteRet, NULL);
					if (isOK)
					{
						printf("%02X-%02X-%02X-%02X-%02X-%02X\n",
							ucData[0], ucData[1], ucData[2],
							ucData[3], ucData[4], ucData[5]);
					} // if( isOK )
				} // if( hDeviceFile != INVALID_HANDLE_VALUE )
			} // if( SetupDiGetDeviceInterfaceDetail() )

			free(DeviceInterfaceDetailData);

		} // for( )

		SetupDiDestroyDeviceInfoList(hDevInfo);

		return TRUE;
	}



	int _tmain(int argc, _TCHAR* argv[])
	{
		TCHAR LocalComputer[MAX_PATH];
		DWORD Size = MAX_PATH - 2;

		GetComputerName(LocalComputer + 2, &Size);
		LocalComputer[0] = _T('\\');
		LocalComputer[1] = _T('\\');

		CONFIGRET cr;
		cr = CM_Connect_Machine(LocalComputer, &m_hMachine);
		DEVNODE dnRoot;
		CM_Locate_DevNode_Ex(&dnRoot, NULL, 0, m_hMachine);

		DEVNODE dnFirst;
		CM_Get_Child_Ex(&dnFirst, dnRoot, 0, m_hMachine);

		RetrieveSubNodes(dnRoot, NULL, dnFirst);

		CM_Disconnect_Machine(m_hMachine);

		GetMacAddress();
		return 0;
	}

##参考
1. <http://blog.csdn.net/uncledou/article/details/8693462>
1. <http://www.codeproject.com/Articles/6445/Enumerate-Installed-Devices-Using-Setup-API>