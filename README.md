Bu uygulamada hem ışınlara hem de üçgenlemeye ihtiyaç vardır. Üzerinde inceleme yapmak için cepheleri yorumlanacak çokgen gereklidir. Bu çokgenin köşe noktaları tespit edilip belirlenen gözlem noktasından ışınlar gönderilir. Bu belirlenen gözlem noktasını hareketli hale getirebilmek için “commoninfo” ile tanımlanır.

![image](https://user-images.githubusercontent.com/114474881/223272237-d5544d91-c988-450c-8e56-851ab805ec39.png)

Şekil 1: 2 Boyutlu şekil

İki boyutlu şeklin görünmeyen cephesinde çok fazla girinti ve çıkıntı varsa bu algoritmayı yavaşlatacak, gereksiz işlem yoğunluğu oluşturacaktır. Bu sebeple çokgenin tüm köşe koordinatları ve gözlem noktası kullanılarak “Spatial > Buffer > Convex Hull ile convexhull oluşturulur. Daha sonra convexhull ın içerisinden çokgenin geometrisi çıkarılır. Ortaya kompleks bir şekil çıkar. Çıkan bu şekil parçalara ayrılır. Daha sonra oluşan çokgenlerden gözlem noktasını içermeyenler silinir. Böylece algoritmayı yavaşlatacak olan bu sorun indirgenmiş olur ve geriye sadece çokgenin incelenecek görünen cepheleri kalır.

![image](https://user-images.githubusercontent.com/114474881/223272898-5a527818-b772-463e-b1ae-39fcb228ffa0.png)

Şekil 2: Girintisi ve çıkıntısı çok olan şekil

![image](https://user-images.githubusercontent.com/114474881/223273222-a57eb832-66e5-4371-9dd7-9d801b168c56.png)

Şekil 3; Çokgenin köşe noktaları ve gözlem noktası ile oluşturulan convex hull

Kesişim analizi ve temas analizi her cephe için yapılır. Gözlem noktasından gönderilen ışınlar her iki köşe noktasında da hiçbir engele rastlamadan hedefe ulaşabiliyorsa bu kenar görünen kenardır. Bunun için önce iki noktadan geçen doğrunun eğimi ayrı ayrı düşünülürse:

m1=(y2-y1)/(x2-x1)     (1)

m2=(y4-y3)/(x4-x3)     (2)

Bu formüller ile de kesişim noktası elde edilir.

x=(m1*x1-m2*x3+y3-y1)/(m1-m2)     (3)

y=m1*(x-x1)+y1     (4)

Ayrıca gözlem noktası ile bina köşeleri arasında oluşturulan üçgen bina köşesi içeriyorsa o köşe noktasındaki gözlem noktasından gönderilen ışın uzatılır ve iki doğrunun kesişimi formülü kullanılarak kesişen kısım elde edilir. Elde edilen kesişim noktası ile görünen köşe noktasını birleştiren doğru cephenin gözlem noktasından görünen kısmıdır. Cephenin diğer kısmı olan görünmeyen köşe noktası ile kesişim noktasının oluşturduğu kısım da cephenin gözlem noktasından görünmeyen kısmıdır.

Farklı bir durum olan iki köşesi de görünmeyen bir cephenin orta kısımlarından bir bölümünün gözlem noktasından görülebilmesidir. Bu alanların tamamı için üçgen içerisinde kalan köşe noktalarına gözlem noktasından gönderilen ışınlar bir nokta veya geometri ile temas etmiyorsa uzatılır eğer bir nokta veya geometri ile temas ediyorsa uzatılmaz. Gözlem noktasından üçgen içerisinde bulunan noktalara gönderilip uzatılan ışının ulaştığı doğru ile kesişimleri bulunur. Elde edilen iki kesişim noktasının oluşturduğu doğru gözlem noktasından cephenin görünen kısmını oluşturur.
 
![image](https://user-images.githubusercontent.com/114474881/223275053-216f9514-d621-43b5-89eb-a51d59a9cc50.png)

Şekil 5: Algoritma

Problemi çok basit düşünerek çıkan diyagram Şekil 5 te görülmektedir. Tabi ki problemin içine girdikçe karşımıza çok daha fazla problem çıkmaktadır ve bunu giderilmeye çalışılmıştır.

MapBsic IDE .mb dosyası aşağıdaki şekildedir:

Include "MapBasic.def"

Declare Sub Main
Declare Sub cokgen_parametre
Declare Sub cokgen_ucgenle
Declare Sub harita_penceresini_aktiflestir
Declare Function tablo_acik_mi(ByVal tabloAdi as String) as Logical
Declare Function ucgen_olustur_dogrula(ckgn_id as Smallint, noktano2 as Smallint, noktano3 as Smallint, ByVal KNoktaTablo as String) as Logical
Declare Sub cikis

Sub Main

' Koordinat sistemi ayarlamasi yapilir.
Set Map Area Units "sq m" Distance units "m"
Set coordsys window frontwindow()

' Menü olusturulur.
Create Menu "COKGEN CEPHE" as

	"&Cokgen Parametreleri" calling cokgen_parametre,
	"(-",
	"Cikis" calling cikis
	
Alter Menu Bar Add "COKGEN CEPHE"

Create Menu "UCGENLEME" as

	"&Cokgen Ucgenle" calling cokgen_ucgenle,
	"(-",
	"Cikis" calling cikis
	
Alter Menu Bar Add "UCGENLEME"

Create Menu "KESİSİM BULMA"

	"&Kesisim Bulma" calling kesisim_bulma
	"(-",
	"Cikis" calling cikis

End Sub

Sub cokgen_parametre
	
Dim n_tablo,j,c_id,n_cokgen,i As Smallint
Dim c_obj As Object
Dim c_perim,c_area,x(1),y(1),s,min_s,max_s,toplam_s,ort_s As Float

'Tabloyu silmek icin prosedür.
Delete from NOKTA_KOOR
Commit Table NOKTA_KOOR
Pack Table NOKTA_KOOR Graphic Data

'Tablo olusturulur.
Select * from COKGEN into cokgen_temp

Fetch first from cokgen_temp

n_tablo = TableInfo(cokgen_temp,TAB_INFO_NROWS)

'Mesaj penceresi silinir.
Print Chr$(12)

Print "Cokgen Cephe"
Print ".........................."

j=1

Do While not eot(cokgen_temp)

'Cokgenin ID si ve geometrisi alinir.
		c_id = cokgen_temp.CokgenID
		c_obj = cokgen_temp-obj
		
'n point seklinde nokta sayisi tanimlanir ve ObjectInfo nokta sayisini 1 fazla döndürdüğü için 1 çikarlir.
		n_cokgen = ObjectInfo(c_obj,OBJ_INFO_NPNTS)-1
		
'kenar hesabinda ilk noktanin tekrar kullanilmasi gerektiği için 1 eklenir.	
		Redim x(n_cokgen+1)
		Redim y(n_cokgen+1)
		
'Cokgenin cevresini bulmak icin fonksiyon.
		c_perim = CartesianPerimeter (c_obj, "m")
		
		For i = 1 to n_cokgen
	
'Cokgenin köse noktalarinin koordinatlari elde edilir.
		x(i) = ObjectNodeX(c_obj,1,i)
		y(i) = ObjectNodeY(c_obj,1,i)
	
'Köse noktalarinin koordinatlari tabloya yazdirilir, ilaveten köşe noktalarinda noktalar olusturulur.
		Insert into NOKTA_KOOR(CokgenID,NoktaNo,Saga_X,Yukari_Y,obj)Values(c_id,i,x(i),y(i),CreatePoint(x(i),y(i)))
		
		Next
		
		toplam_s = 0
		
		For i = 1 to n_cokgen
		
			if i = n_cokgen then
			
'Kenar bulmada ilk kenar tekrar kullanılacağı icin 1 eklenir.
				x(i+1) - x(1)
				y(i+1) - y(1)
				
			End if
			
			s = CartesianDistance(x(i),y(i),x(i+1),y(i+1),"m")
		
'min max kenar degerleri sürekli güncellenir.
			If i = 1 then
				min_s = s
				max_s = s
			Else
				If (s < min_s) then min_s = s
				ElseIf (s > max_s) then max_s = s
				End If
			End If
			
			toplam_s = toplam_s + s
			
		Next
		
'ortalama kenar hesaplanir.
		ort_s = Round(toplam_s / n_cokgen,0.01)
		
		Update COKGEN
		
			Set
			
'Cokgenle ilgili bilgiler elde edilir. RowID deki satir numarasine göre yazilir, j sayaci ile degistirilir.
				CokgenID = j,
				NoktaSayisi = n_cokgen,
				Cevre = c_perim,
				MinKenarUz = min_s,
				MaxKenarUz = max_s,
				OrtKenarUz = ort_s,
		Where RowID = j
	
'j sayac görevi görür cokgen ID ye bu deger atanır.
	j = j + 1
	
'sonraki satira gecilir.
	Fetch Next From cokgen_temp
	
Loop

'Tablolar acilir.
Browse * From COKGEN

Browse * From NOKTA_KOOR

Print "İSLEM TAMAMLANDI!"

End Sub

Sub kesisim_bulma

Dim n_tablo,j,c_id,n_cokgen,i As Smallint
Dim c_obj As Object
Dim c_perim,c_area,x(1),y(1),s,min_s,max_s,toplam_s,ort_s As Float

L1 = Line joining points A(x1,y1) and B(x2,y2) L2 = Line joining points c(x3,y3) and D(x4,y4)

	For Line L1

		Line Equation : y = m1*x + c1

		slope m1 : (y2-y1)/(x2-x1)

		Y intercept : c1 = (y1 - m1*x1)

	For Line L2

		Line Equation : y = m2*x + c2

		slope m2 : (y4-y3)/(x4-x3)

		Y intercept : c2 = (y3 - m2*x3)

	For Point of Intersection

		Solving the above equations we get

		x = (c2 -c1)/(m1-m2)

		y = (c1*m2 - c2*m1)/(m2-m1)

End sub

Sub cokgen_ucgenle

Dim n_tablo,j,c_id,n_cokgen, i As Smallint
Dim c_obj As Object
Dim c_perim,c_area,x(1),y(1),s,min_s,max_s,toplam_s,ort_s As Float
Dim CokgenTablo,KNTablo,nno1,nno2,nno3 as String
'Bu varyans degiskeni ile tablodaki isimler yerine böyle takma isimler kullanilabilir.
Dim KNStn11, KNStn12, KN1_RowOrder, KN1ID, KN2ID, KN1KoorX, KN1Obj, KN2Obj, CKGNID, CKGNObj, KNStnX, KNStnY as Alias
Dim ucgen_kontrol as Logical

'Alt prosedür cagirilir.
Call harita_penceresini_aktiflestir

KNTablo = "KOSENOKTA"

Delete from KNTablo
Commit Table KNTablo
Pack Table KNTablo Graphic Data

'Tablonun acik mi kapali mi oldugu kontrol edilir.
If tablo_acik_mi("KN1Tablo")then
	Close table "KN1Tablo"
End if

If tablo_acik_mi("KN2Tablo")then
	Close table "KN2Tablo"
End if

CokgenTablo = "COKGENTABLO"

'Gecici bir tabloya deger atamasi yapilir.
Select * from CokgenTablo into cokgen_temp

Fetch first from cokgen_temp

n_tablo = TableInfo(cokgen_temp,TAB_INFO_NROWS)

Print Chr$(12)

Do While not eot(cokgen_temp)

	c_id = cokgen_temp.COKGENID
	c_obj = cokgen_temp.obj
	
	n_cokgen = ObjectInfo(c_obj, OBJ_INFO_NPNTS)-1
	
	Redim x(n_cokgen+1)
	Redim y(n_cokgen+1)
	
	c_perim = CartesianPerimeter(c_obj,"m")
	
	c_area = CartesianArea(c_obj,"sq m")
	
	For i = 1 to n_cokgen
	
		x(i) = ObjectNodeX(c_obj,1,i)
		y(i) = ObjectNodeY(c_obj,1,i)
		
'Köse koordinatlari yazdirilir.
		Insert into KNTablo(CpkgenID,NoktaNo,Saga_X,Yukari_Y,obj)Values(c_id,i,x(i),y(i),CreatePoint(x(i),y(i)))
		
	Next
	
	If tablo_acik_mi("KN1Tablo")then
		Close table "KNTablo"
	End if
	
	If tablo_acik_mi("KN2Tablo")then
		Close table "K2Tablo"
	End if
	
'2 ayri tablo olusturulur.
	Select NoktaNo "ID1",Saga_X,Yukari_Y From KNTablo Order By Saga_X Into KN2TabloX
	Commit table KN1TabloX as "KN1Tablo.tab"
	Open table "KN1Tablo"
	
	Select NoktaNo "ID2",Saga_X,yukari_Y from KNTablo Order By Saga_X Into KN2TabloX
	Commit table KN2TabloX as "KN2Tablo.tab"
	Open table "KN2Tablo"
	
'Varyans degiskenleri
	KN1ID = KN1Tablo & ".ID1"
	KN2ID = KN2Tablo & ".ID2"
	KN1KoorX = KN1Tablo & ".Saga_X" 'X'e göre sirralama icin
	KN1obj = KN1Tablo & ".obj"
	KN2obj = KN2Tablo & ".obj"
	CKGNID = CokgenTablo & ".CokgenID"
	CKGNObj = CokgenTablo & ".Obj"
	
	Select CKGNID,KN1ID,KN2ID, CartesianObjectDistance(KN1obj,KN2obj,"m")"Uz_m",KN1KoorX
	From KN1Tablo,CokgenTablo,KN2Tablo
	Where KN1obj Within CKGNObj And KN2obj Within CKGNObj and KN1ID <> KN2ID
	Order by Col5, Col4
	Into UZUNLUK
	
	Browse * From UZUNLUK
	
	Map from UZUNLUK, CokgenTablo
	
	Fetch first from UZUNLUK
	
	Do While Not EOT(UZUNLUK)
	
		nno1 = UZUNLUK.ID1
		nno2 = UZUNLUK.ID2
		
		Fetch Next From UZUNLUK
	
		nno3 = UZUNLUK.ID2
		
		ucgen_kontrol = ucgen_olustur_dogrula (c_id as Smallint, nno2 as Smallint, nno3 as Smallint, ByVal "KNTablo" as String) as Logical
	
	Loop
	
	Fetch next from cookgen_temp
	
Loop

End Sub

Function Ucgen_olustur_dogrula (ckgn_id as Smallint, noktano1 as Smallint, noktano2 as Smallint, nooktano3 as Smallint, ByVal KNoktaTablo as String) as Logical

Dim ucgen_nesne as Object
Dim x_u, y_u as Float

	Create Region Into Variable ucgen_nesne 0
	
	Select * From KNoktaTablo Where NoktaNo = Any (noktano1, noktano2, nooktano3) Into ucgen_nokta
	
	Fetch First From ucgen_nokta
	
	For j = 1 to 3
	
		x_u = ucgen_nokta.SagaX
		y_u = ucgen_nokta.SagaY
		
		Alter object ucgen_nokta
		
	Next
	
	Insert Into UCGEN (Object) Values (ucgen_nesne)
	
End function

Sub cikis

'Cikis yaparken olusan tablolar sildirilir.
	Print Chr$(12)
	
	Delete from NOKTA_KOOR
	Commit Table NOKTA_KOOR
	Pack Table NOKTA_KOOR Graphic Data
	
	End Program
	
 End Sub

