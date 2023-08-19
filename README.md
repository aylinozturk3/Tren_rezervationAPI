# Tren_rezervationAPI
from flask import Flask, request, jsonify  #Flask kütüphanesinin import edilmesi
import requests
url="http://localhost:5000/reservation_kontrol"
data={
    "Tren":{
        "Ad": "Başkent Ekspress",
        "Vagonlar":[
            {"Ad": "Vagon 1", "Kapasite":100, "DoluKoltuksayisi":68},
            {"Ad": "Vagon 2", "Kapasite":90, "DoluKoltuksayisi":50},
            {"Ad": "Vagon 3", "Kapasite":80, "DoluKoltuksayisi":80}
        ]
    }
}
response=requests.post(url,json=data)
app=Flask(__name__) #Flask uygulamasını oluştururken kullanılan bir Python kod satırıdır.
@app.route("/reservation") #Flask uygulamasında belirli bir URL yolunu belirtir
def reservation(traininfo, peoplenum, allow_diff_vagons): #Rezervasyon fonksiyonu
    #URL'den gelen parametler alınır
    traininfo= request.args.get("traininfo")
    peoplenum=request.args.get("peoplenum")
    allow_diff_vagons = request.args.get("allow_diff_vagons")
    print("Parameters received successfully")
    availablevagons=[]
    for vagon in traininfo["Vagonlar"]: #vagonları tek tek dolaşıyor bu döngüde
        occupancy=vagon["Dolukoltuksayisi"] / vagon["Kapasite"] #doluluk oranı hesaplanması
        if occupancy < 0.7:
             availablevagons.append(vagon)
        else:
            availablevagons.append([])

    if not availablevagons:
           return []

    if not allow_diff_vagons:
        total_= sum(vagon["Kapasite"] for vagon in availablevagons) #müsait vagonlardaki toplam kapasitenin hesaplanması
        if peoplenum <= total_:
             assigned_vagons= [{"Vagon_Adi:": availablevagons[0]["Ad"],"KisiSayısı:":peoplenum}]
             return assigned_vagons #yerleştirildikleri vagonları gösterir

    assigned_vagons= [] #bir kez daha boş küme atanır ki diğer kişileri yerleştikleri vagonu gösterirken karışıklık olmasın
    kalankisi=peoplenum

    for vagon in availablevagons:
        if kalankisi > 0:
            if vagon["Kapasite"]>= kalankisi:
                assigned_vagons.append({"VagonAdi:":vagon["Ad"], "KisiSayisi:":kalankisi})
                kalankisi =0


     return assigned_vagons

@app.route("/reservation_kontrol ",methods=["POST"]) #HTTP POST isteği alındığında URL yoluyla ilişkilendirilmiş bir işlevi tanımlar
def reservation_kontrol(): #Rezervasyon kontrol fonksiyonu
        data= request.get_json()  # Gelen JSON verisini alır

        #Aşağıda, "traininfo","peoplenum","allow_diff_vagons" degişkenleri bir JSON verisinden gelen değerlere atanıyor
        traininfo=data["Tren"]
        peoplenum=data["RezervasyonYapacakKisiSayısı"]
        allow_diff_vagons=data["KisilerFarkliVagonlaraYerleştirilebilir"]

        possibilityof_reservation, detailsof_placement= reservation( traininfo,peoplenum,allow_diff_vagons) #yukarıkdaki fonksiyon kullanılıyor

        response = {    #Alınacak çıktı
            "RezervasyonYapılabilir":possibilityof_reservation,
            "YerlesimmAyrintisi":detailsof_placement
        }
        print("Data processed successfully")
        return jsonify(response) #jsonify, JSON verilerini Flask'in anlayabileceği bir formata dönüştürür


if __name__ == '__main__' :  #Flask uygulamasını çalıştırmak için kullanılır
      app.run()
