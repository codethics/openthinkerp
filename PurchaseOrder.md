#Purchase Order

DRAFT!

# Pendahuluan #

Catatan kecil bagaimana PO ditangani di OpenThink ERP


# Details #

Pembelian Barang

Jika ingin membeli barang, yang pertama harus dilakukan adalah membuat purchase order. Pilihlah
Menu Purchases → Purchase Order Entry.

Gambar 1. Halaman Awal Purchase Order Entry

Pilih dan tambah Item yang ingin dibeli, tulis memo bila perlu.

Gambar 2. Halaman Purchase Order Entry, ketika sudah ada item yang ditambahkan

Jika semua item yang ingin dipesan sudah dimasukkan, klik saja tombol Place Order.

Ketika Anda mengklik tombol Place Order maka, data item pesanan akan dimasukkan ke tabel purch\_orders dan purch\_order\_details, perintah SQL yang berhubungan adalah sebagai berikut :

INSERT INTO purch\_orders (supplier\_id, Comments, ord\_date, reference, requisition\_no, into\_stock\_location, delivery\_address) VALUES(........) ;

INSERT INTO purch\_order\_details (order\_no, item\_code, description, delivery\_date,	unit\_price,	quantity\_ordered) VALUES (........)

Sesudah meng-klik Place Order, Anda akan dibawa ke halaman informasi yang memberitahukan Anda jika Purchase Order telah dibuat.
















Gambar 3.


Jika View Purchase Order di klik, Anda akan melihat detail data pemesanan yang baru saja anda lakukan :


Gambar 4

Dari informasi diatas bisa dilihat Quantity Received dan Quantity Invoiced masih 0.

Selanjutnya, seperti terlihat dari Gambar 3, Anda bisa meng-klik Receive Items on This Purchase Order  jika Anda telah menerima barang pesanan Anda atau memilih Select An Outstanding Purchase Order, untuk melihat semua Outstanding Purchase Order yang telah dilakukan.


> Gambar 5.

Dari  Search Outstanding dan Purchase Order ini Anda bisa meng-edit Purchase Order, men-cetak  serta mengkonfirmasi penerimaan barang dengan meng-klik Receive.

> Gambar 6

Jika meng-klik Receive, akan tampil halaman seperti pada Gambar 7 :








Gambar 7

Isilah kolom This Delivery sesuai dengan jumlah barang yang diterima (asalkan jangan melebihi outstanding tentunya ...), jika sudah, klik tombol Process Receive Items.

Dibelakang layar, ketika Anda mengklik tombol Process Receice Items, OpenThink akan mengeksekusi fungsi add\_grn() yang menambahkan data pada tabel grn\_batch kemudian mengupdate average material cost dengan rumus :

> $material\_cost = ($qoh **$material\_cost + $qty** $price\_in\_home\_currency) /	$qoh;

$qoh adalah :

"SELECT SUM(qty) FROM stock\_moves
> WHERE stock\_id='$stock\_id'
> AND tran\_date <= '$date'";

> if ($location != null) {
> > $sql .= " AND loc\_code = '$location'";

> }

seperti dapat dilihat pada lib/db/inventory\_db.inc.php

Material cost di update dengan Sql berikut :

$sql = "UPDATE stock\_master SET material\_cost=".db\_escape($material\_cost)."
> WHERE stock\_id='$stock\_id'"


Kemudian terakhir akan dieksekusi dua fungsi berikut  :

$grn\_item = add\_grn\_detail\_item($grn, $order\_line->po\_detail\_rec,
> $order\_line->stock\_id, $order\_line->item\_description,
> $order\_line->standard\_cost,	$order\_line->receive\_qty, $order\_line->price);

> /**Update location stock records - NB  a po cannot be entered for a service/kit parts**/
> add\_stock\_move(25, $order\_line->stock\_id, $grn, $location, $date_, "",
> $order\_line->receive\_qty, $order\_line->standard\_cost,
> $po->supplier\_id, 1, $order\_line->price);_


Atau singkatnya ketika tombol Process Receive Items di klik inilah yang terjadi :

add\_grn() → add\_grn\_batch() → update\_average\_material\_cost() → add\_grn\_detail\_item() → add\_stock\_move()

Fungsi
Keterangan
add\_grn()
Fungsi Utama yang dipanggil ketika Process Receive Items dilakukan
add\_grn\_batch()
Memasukkan data PO ke tabel grn\_batch :

“INSERT INTO grn\_batch (purch\_order\_no, delivery\_date, supplier\_id, reference, loc\_code)
> VALUES(......)”
update\_average\_material\_cost()
update average material cost
add\_grn\_detail\_item()
Mengupdate tabel purch\_order\_details dan menambah data item yang dibeli ke grn\_items
add\_stock\_move()
Update location stock records - NB  a po cannot be entered for a service/kit parts


TODO : Script diatas nanti akan dijelaskan dengan kata-kata ...

Setelah Process Receive Items di klik maka Anda akan dibawa ke halaman konfirmasi seperti terlihat pada Gambar 8.


Gambar 8

Jika View this Delivery di klik maka akan terlihat Gambar 9, halaman ini memperlihatlan detail data dari Purchase Order Delivery. Dapat kita lihat, disini PO Delivery ini belum dibuat Invoice nya ..

Gambar 9

Perlu diketahui, sebelum dibuat Invoice, MAKA belum ada transaksi yang dicatat kedalam GL. Pada Gambar 8, jika View GL Journal Entries for this Delivery Kita klik, maka tidak ada data GL yang tercatat, seperti terlihat pada Gambar 10.


Gambar 10.

Untuk membuat Invoive PO ini Anda bisa meng-klik Menu Purchases → Supplier Invoice. Maka Anda akan melihat halaman seperti pada Gambar 11.

Gambar 11.

Pada Gambar 11 terlihat ada 1 buah Items Received Yet to be Invoiced.

Kolom delivery jika di klik akan menampilkan informasi detail Purchase Order Delivery seperti terlihat pada Gambar 9. Sedangkan jika nilai pada kolom P.O diklik maka  akan menampilkan informasi detail Purchase Order tersebut, seperti terlihat pada Gambar 12.



Gambar 12

Edit Qty To Invoice dan Order Price bila perlu (Gambar 11), kemudian klik Add. Untuk GL Items for Invoice ..., untuk sekarang masih belum bisa dijelaskan ... :( . tolong diskusikan dengan akuntan Anda ...


Ketika tombol Enter Invoice di klik, maka OpenThink ERP akan memproses Invoice Anda, dibelakang layar, OpenThink ERP akan mengeksekusi fungsi 	handle\_commit\_invoice().Alur singkatnya adalah seperti ini :

handle\_commit\_invoice() → add\_supp\_invoice() → add\_supp\_trans() → add\_gl\_trans\_supplier()  → add\_supp\_invoice\_gl\_item()


Inti business prosess nya berada pada fungsi add\_supp\_invoice() .., ini aka diterangkan nanti ...


Gambar 13

Jika sudah klik tombol Enter Invoice. Anda akan dibawa ke halaman seperti pada Gambar 14.

Gambar 14

Jika View Invoices di klik akan muncul halaman seperti pada Gambar 15.

> Gambar 15

Jika  View the GL Journal Entries for this Invoice di klik akan muncul halaman seperti pada Gambar 16.


> Gambar 16.

Jika kita melihat pada entri GL di Gambar 16, dapat dilihat kalau Kita masih memiliki Utang  Dagang sebesar 100.000,- akibat pembelian barang ini. Lalu bagaimana cara membayar Utang akibat pembelian ini ?

Untuk melakukan pembayaram, pilihlah menu Purchases → Payments to Suppliers (Payments).

Gambar 17.

Jika form Supplier Payment Entry sudah selesai kita buatm klik Enter Payment.

Ketika tombol Enter Payment di klik, OpenThink ERP akan mengeksekusi fungsi : handle\_add\_payment()  yang akan mengeksekusi  add\_supp\_payment() yang didalam fungsi ini ia mengeksekusi fungsi  add\_supp\_trans() yang membuat entri baru di tabel supp\_trans untuk pembayaran supplier dan kemudian  add\_gl\_trans\_supplier() , yang mendebit account kreditur dalam hal ini adalah Supplier dengan payment + diskon , kemudian fungsi add\_bank\_trans() dieksekusi untuk  memasukkan entri di bank\_entry.

Setelahnya, Anda akan dibawa ke halaman seperti Pada Gambar 18.

Gambar 18.

Jika View GL Entries for this Payment di Klik, akan muncul halaman seperti pada Gambar 19.

Gambar 19

Kita lihat, pada GL, Utang Dagang telah ter-debit dan Kas di credit.

Kemudian untuk mengalokasikan Payment ini, tinggal klik Allocate this Payment pada Gambar 19 atau melalui menu Purchases → Allocate Supplier Payments or Credit Notes. Anda Akan melihat halaman seperti pada Gambar 20.
Gambar 20

Pada Gambar 20, Anda lihat ada Supplier Payment sebesar 100.000 yang belum dialokasikan. Jika nilai di kolom # di klik, maka akan muncul detail dari Payment to Supplier ini, seperti terlihat pada Gambar 21.

Gambar 21

Untuk melakukan alokasi pembayaran ini, tinggal klik Allocate pada Gambar 20. Anda akan dibawa ke halaman Allocate Supplier Payment or Credit Note seperti pada Gambar 22.



Gambar 22.

Tentukan Alokasi untuk Transaksi ini, kemudian klik Process. Jika kita melihat pada Supplier Inquiry dengan Tipe Payment maka data transaksi diatas sudah terlihat, seperti terlihat pada Gambar 23.


Gambar 23