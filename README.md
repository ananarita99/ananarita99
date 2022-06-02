#import librari
import json
from bs4 import BeautifulSoup as bp
import requests as rq
URL = "https://www.emag.ro/telefon-mobil-samsung-galaxy-s10e-dual-sim-128gb-6gb-ram-4g-black-sm-g970fzkdrom/pd/DTY7QLBBM/"
page = rq.get(URL)

siteWeb = bp(page.content, 'html.parser') # parsare site din link intr-un obiet  beatifulsoup

#Verifica daca cuvantul "rated-" exista in obiectul tag luat din div review
#Pe emag review-ul este afisat sub forma de stelute, iar numarul lor depinde de rated-1 rated-2 rated-3 rated-4 rated-5
def extragereReview(review):
    if "star-rating star-rating-read rated-5 star-rating-sm" in str(review):
        return "5 stele"
    elif "star-rating star-rating-read rated-4 star-rating-sm" in str(review):
        return "4 stele"
    elif "star-rating star-rating-read rated-3 star-rating-sm" in str(review):
        return "3 stele"
    elif "star-rating star-rating-read rated-2 star-rating-sm" in str(review):
        return "2 stele"
    elif "star-rating star-rating-read rated-1 star-rating-sm" in str(review):
        return "1 stele"
    else:
        return "fara stele"

def reviewEmag():
    # ia toate informatiile din sectiunea de review
    reviewComentarii =  siteWeb.find_all("div", class_="product-review-item js-review-item row")
    # lista in care o sa fie salvate fiecare dictionar cu datele de pe site
    listDictReview = []
    # parcurge toate review-urile care sunt in lista de review
    for reviewIndividual in reviewComentarii:
        # extragem titlul din elemtul html de dimensiune h3 cu clasa respectiva
        titluReview = reviewIndividual.find("h3", class_="product-review-title").find("a").getText()
        # extragem scor-ul din sectiunea div clasa respectiva
        scorReview = reviewIndividual.find("div", class_="star-rating-container mrg-btm-xs")
        scor = extragereReview(scorReview)
        # extragem autorul din elementul paragraf din clasa
        autor = reviewIndividual.find("p", class_="product-review-author mrg-top-xs semibold").getText()
        # extragem data din elementul paragraf
        data = reviewIndividual.find("p", class_="small text-muted mrg-sep-none").getText()
        # extragem comentariu din sectiunea div
        comentariu = reviewIndividual.find("div", class_="mrg-btm-xs js-review-body review-body-container").getText()
        #dictionar predefinit
        informatiiReview = {
            "titlu": titluReview,
            "autor": autor,
            "data": data,
            "comentariu": comentariu,
            "rating": scor
        }
        #exit(1)
        #printare dictionar pentru testare
        print(json.dumps(informatiiReview, sort_keys=True, indent=4))
        listDictReview.append(informatiiReview)
    #print(listDictReview)
    #folosim metoda with open sa scriem in fisier deoarece se inchide automat la finalizarea scrierii in el
    with open("reviewEmag.json", "w") as write_file:
        # indentam cu 4 ca sa arate frumos scris in fisier
        json.dump(listDictReview, write_file, indent=4)

reviewEmag()


# Scriere in json https://www.w3schools.com/python/python_json.asp
# Extragere date din website model https://stackoverflow.com/questions/65095206/extract-date-from-multiple-webpages-with-python
