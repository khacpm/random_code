# Add enchant rate to Lab

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

What things you need to edit is just 2 files

```
InfoGenDlg
CityLab
```

## Step 1 - Generate omi.tex with enchant rate
Open InfoGenDlg, find OnButtonIgAll function
Put below code under ItemInfo block

```
//enchant info
dbType = DB_ENCHANT_INFO;
ez_map<INT, ENCHANT_INFO> mapEnchantInfo;
nObjects = CAtumDBHelper::LoadEnchantInfo(&odbcStmt, &mapEnchantInfo);
if (0 >= nObjects)
{
	MessageBox("LoadEnchantInfo error from DB!!, Please check DB Schema.");
	return;
}
nObjects = mapEnchantInfo.size();
WriteFile(hFile, (LPCVOID)&dbType, sizeof(DB_TYPE), &dwBytesWritten, NULL);
WriteFile(hFile, (LPCVOID)&nObjects, sizeof(int), &dwBytesWritten, NULL);
ez_map<INT, ENCHANT_INFO>::iterator itrEnchantInfo = mapEnchantInfo.begin();
const int sizeOfEnchant_INFO = sizeof(ENCHANT_INFO);
while (itrEnchantInfo != mapEnchantInfo.end())
{
	ENCHANT_INFO *cell = &itrEnchantInfo->second;
	WriteFile(hFile, (LPCVOID)&cell, sizeOfEnchant_INFO, &dwBytesWritten, NULL);
	itrEnchantInfo++;
}
```

Add new Element to enums DB_TYPE (AtumParam)

```
DB_ENCHANT_INFO
```

## Step 2 - Load omit.text with enchant rate to Memory

Open AtumDatabase
Modify InitDeviceObjects function
Add below code into “switch (header.nType)” block

```
case DB_ENCHANT_INFO:
{
	if (!LoadEnchantData(fd, header.nDataCount))
	{
		DBGOUT("Data File Read Error(omi.tex : DB_ITEM)\n");
		return;
	}
}
	break;
```

Add new Function into AtumDatabase 

```
BOOL CAtumDatabase::LoadEnchantData(FILE* fd, int nCount)
{
	ASSERT_ASSERT(fd);
	const int sizeOfENCHANT_INFO = sizeof(ENCHANT_INFO);
	for (int i = 0; i < nCount; i++)
	{
		ENCHANT_INFO * cell = new ENCHANT_INFO;
		if (fread(cell, sizeOfENCHANT_INFO, 1, fd) == 0)
			return FALSE;
		m_mapEnchantInfoTemp[cell->EnchantItemNum] = cell;
	}
	return TRUE;
}

ENCHANT_INFO* CAtumDatabase::GetEnchantInfoByEnchantItemNum(INT nEnchantItemNum)
{
	ENCHANT_INFO* pRtnEnchantInfo = NULL;
	CMapEnchantInfoIterator itInfoEnchantTemp = m_mapEnchantInfoTemp.find(nEnchantItemNum);
	if (itInfoEnchantTemp == m_mapEnchantInfoTemp.end())
	{
		return NULL;
	}
	pRtnEnchantInfo = (itInfoEnchantTemp->second);
	return pRtnEnchantInfo;
}
```

Add new Static Vector of EnchantInfo into AtumDatabase.h

```
	CMapEnchantInfoList m_mapEnchantInfoTemp;
```

Add New TypeDef (TypeDef.h)

```
typedef map<int, ENCHANT_INFO*> CMapEnchantInfoList;
typedef map<int, ENCHANT_INFO*>::iterator CMapEnchantInfoIterator;
```

## Step 3 - Modify CityLab

Add new CityLab function

```
void CINFCityLab::UpdatePriceText()
{
	if (m_SuccessRate > 0)
	{
		float trueRate = m_SuccessRate + (m_SuccessRateBonus * m_SuccessRate * 0.01f);
		sprintf(m_TextPrice, "(\\g%.0f%%\\g) %d", trueRate, m_szPrice);
	}
	else
		sprintf(m_TextPrice, "%d", m_szPrice);
}
```

Modify or replace InvenToSourceItem(...) with this code:

```
void CINFCityLab::InvenToSourceItem(CItemInfo* pItemInfo, int nCount)
{
	switch (pItemInfo->Kind)
	{
		case ITEMKIND_ENCHANT:
		case ITEMKIND_GAMBLE:
		{
			m_CurrentEnchantItemNum = pItemInfo->ItemNum;
			ENCHANT_INFO* enchantInfo = g_pDatabase->GetEnchantInfoByEnchantItemNum(pItemInfo->ItemNum);
			if (enchantInfo != NULL)
			{
				SetPrice(enchantInfo->EnchantCost);
#ifdef C_KHACPM_DEBUG
				char debug_text[128];
				sprintf(debug_text, "[DEBUG] EnchantNum: %d, Cost: %d", pItemInfo->ItemNum, enchantInfo->EnchantCost);
				g_pD3dApp->m_pChat->CreateChatChild(debug_text, COLOR_ERROR);
#endif
			}
		}
		break;

		case ITEMKIND_PREVENTION_DELETE_ITEM:
		{
			if (pItemInfo->ItemInfo->ArrDestParameter[0] == 192)
			{
				m_SuccessRateBonus = pItemInfo->ItemInfo->ArrParameterValue[0] / 100;
#ifdef C_KHACPM_DEBUG
				char debug_text[128];
				sprintf(debug_text, "[DEBUG] Bonus Rate %.0f%%", m_SuccessRateBonus);
				g_pD3dApp->m_pChat->CreateChatChild(debug_text, COLOR_ERROR);
#endif
			}
		}
		break;
		
		case ITEMKIND_AUTOMATIC:
		case ITEMKIND_VULCAN:
		case ITEMKIND_DUALIST:
		case ITEMKIND_CANNON:
		case ITEMKIND_RIFLE:
		case ITEMKIND_GATLING:
		case ITEMKIND_LAUNCHER:
		case ITEMKIND_MASSDRIVE:
		case ITEMKIND_ROCKET:
		case ITEMKIND_MISSILE:
		case ITEMKIND_BUNDLE:
		case ITEMKIND_DEFENSE:	//armor
		case ITEMKIND_SUPPORT:	//engine
		case ITEMKIND_RADAR:	//radar
		case ITEMKIND_ACCESSORY_UNLIMITED:	//coating or shield generator
		{
			m_CurrentNumOfEnchants = pItemInfo->GetEnchantNumber();
#ifdef C_KHACPM_DEBUG
			char debug_text[128];
			sprintf(debug_text, "[DEBUG] Item'sEnchantNum: %d", m_CurrentNumOfEnchants);
			g_pD3dApp->m_pChat->CreateChatChild(debug_text, COLOR_CHAT_WAR);
#endif
		}
		break;

		default:
		break;
	}

	if (m_CurrentEnchantItemNum != 0 && m_CurrentNumOfEnchants != -1)
	{
		ENCHANT_INFO* enchantInfo = g_pDatabase->GetEnchantInfoByEnchantItemNum(m_CurrentEnchantItemNum);
		if (enchantInfo != NULL)
		{
			m_SuccessRate = enchantInfo->ProbabilityPerLevel[m_CurrentNumOfEnchants] / 100;
#ifdef C_KHACPM_DEBUG
		char debug_text[128];
		sprintf(debug_text, "[INFO] Sum[%d, %d]", m_CurrentEnchantItemNum, m_CurrentNumOfEnchants);
		g_pD3dApp->m_pChat->CreateChatChild(debug_text, COLOR_ERROR);
		sprintf(debug_text, "[INFO] m_SuccessRate: %d", m_SuccessRate);
		g_pD3dApp->m_pChat->CreateChatChild(debug_text, COLOR_ERROR);
#endif
		UpdatePriceText();
		}
	}

	// Inven. It is erased from.
	if (IS_COUNTABLE_ITEM(pItemInfo->Kind))
	{
		ASSERT_ASSERT(pItemInfo->CurrentCount >= nCount);

		vector<CItemInfo*>::iterator it = m_vecSource.begin();
		while (it != m_vecSource.end())
		{
			if ((*it)->UniqueNumber == pItemInfo->UniqueNumber)
			{
				(*it)->CurrentCount += nCount;
				break;
			}
			it++;
		}
		if (it == m_vecSource.end())
		{
			CItemInfo* pNewItem = new CItemInfo((ITEM_GENERAL*)pItemInfo);
			pNewItem->CopyItemInfo(pItemInfo);
			pNewItem->CurrentCount = nCount;
			m_vecSource.push_back(pNewItem);
			UpdateMixPrice();
		}
		g_pStoreData->UpdateItemCount(pItemInfo->UniqueNumber, pItemInfo->CurrentCount - nCount);
	}
	else
	{
		CItemInfo* pNewItem = new CItemInfo((ITEM_GENERAL*)pItemInfo);
		pNewItem->CopyItemInfo(pItemInfo);
		m_vecSource.push_back(pNewItem);
		g_pStoreData->DeleteItem(pItemInfo->UniqueNumber);
		UpdateMixPrice();
	}
}
```

Modify CINFCityLab::Render()
Replace render SPI block with this code:

```
if (m_szPrice > 0)
{
#ifdef C_EPSODE4_UI_CHANGE_JSKIM	
		SIZE sSize = m_pFontPrice->GetStringSize(m_TextPrice);
		m_pFontPrice->DrawText(SPI_START_X - sSize.cx, SPI_START_Y, GUI_FONT_COLOR, m_TextPrice, 0L);
#else
		m_pFontPrice->DrawText(SPI_START_X, SPI_START_Y, GUI_FONT_COLOR, m_szPrice, 0L );
#endif
}
```

Add missing elements to header file

```
INT		m_szPrice;
char		m_TextPrice[64];
INT		m_SuccessRate;
FLOAT		m_SuccessRateBonus;
INT		m_CurrentEnchantItemNum;
INT		m_CurrentNumOfEnchants;
```

Add Initial set in CityLab.cpp

 
```
m_szPrice = 0;
m_SuccessRate = -1;
m_SuccessRateBonus = 0;
m_CurrentEnchantItemNum = 0;
m_CurrentNumOfEnchants = -1;
```

## Known issues: Known issues:
Only show when both enchant item and weapon was added
