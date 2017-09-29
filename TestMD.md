# Add enchant rate to Lab

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

What things you need to edit is just 2 files

```
InfoGenDlg
CityLab
```

### Step 1 - Generate omi.tex with enchant rate
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
```


			
Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc
