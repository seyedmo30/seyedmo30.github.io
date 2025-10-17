
### slice

دقت کنید تو مصاحبه اگر خواستید تو یه اسلایس بریزید ، اگر با توجه با ایندکس اون می خواید بریزید ، باید len , cap اون رو یکی بدید نت با توجه به ایندکس ، اون رو آپدیت کنید ولی اگه می خوایید به اون اپند کنید ، باید len ون رو صفر بزارید


		resultList := make([]int, lenList, lenList)
  		resultList[indexRes] = value


		resultList := make([]int, 0, lenList)
  		resultList =append(resultList, value)

    