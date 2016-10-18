# smnhuda

# masing-masing 28 bit, digunakan sebagai penyimpanan di KS (Key Stuctured) putaran untuk menghasilkan putaran kunci (alias subkey)
private static int[] C = new int[28];
	private static int[] D = new int[28];
	
	 private static String asciiToHex(String asciiValue)
   {
      char[] chars = asciiValue.toCharArray();
      StringBuffer hex = new StringBuffer();
      for (int i = 0; i < chars.length; i++)
      {
         hex.append(Integer.toHexString((int) chars[i]));
      }
      return hex.toString();
   }
   private static String hexToASCII(String hexValue)
   {
      StringBuilder output = new StringBuilder("");
      for (int i = 0; i < hexValue.length(); i += 2)
      {
         String str = hexValue.substring(i, i + 2);
         output.append((char) Integer.parseInt(str, 16));
      }
      return output.toString();
   }
   
# Dekripsi membutuhkan 16 subkunci yang akan digunakan dalam proses yang sama persis enkripsi, dengan satu-satunya perbedaan adalah bahwa tombol yang digunakan dalam urutan terbalik, yaitu kunci terakhir digunakan pertama dan seterusnya. Oleh karena itu, selama enkripsi ketika tombol pertama dihasilkan, mereka disimpan dalam Array. Dalam hal ini kita ingin memisahkan enkripsi dan dekripsi program, maka kita perlu untuk menghasilkan subkunci pertama yang bertujuan menyimpan mereka dan kemudian menggunakannya dalam urutan terbalik.
private static int[][] subkey = new int[16][48];
	
 public static void main(String args[]) {
		System.out.println("====================================================================");
		System.out.println("Masukkan plaintext 8 karakter:");
		String demoString = new Scanner(System.in).nextLine();
    String hexEquivalent = asciiToHex(demoString);
     
# nilai hexadecimal dari original string  
   hexEquivalent = hexEquivalent.toUpperCase();
   System.out.println("Nilai Hexa: "+ hexEquivalent); 
   System.out.println("Nilai Biner:");
      
   int inputBits[] = new int[64];
# inputBits akan menyimpan 64 bit input sebagai int array ukuran 64. Program ini menggunakan array int untuk menyimpan bit, agar lebih sederhana. Agar pemrograman menjadi efisien, gunakan tipe data yang panjang. Tapi meningkatkan kompleksitas Program yang tidak perlu untuk konteks ini.
for(int i=0 ; i < 16 ; i++) {

# Untuk setiap karakter dalam input 16 bit, kita mendapatkan nilai biner dengan terlebih dahulu parsing ke int dan kemudian mengkonversi ke biner string
String s = Integer.toBinaryString(Integer.parseInt(hexEquivalent.charAt(i) + "", 16));

# Java tidak menambahkan padding angka nol, yaitu 5 dikembalikan sebagai 111 tapi java membutuhkan 0111. Oleh karena itu, pada proses looping sementara ini menambahkan padding 0 terhadap nilai biner.
while(s.length() < 4) {
				s = "0" + s;
			}
      
# Tambahkan 4 bits yang sudah kita ekstrak ke bit array
for(int j=0 ; j < 4 ; j++) {
				inputBits[(4*i)+j] = Integer.parseInt(s.charAt(j) + "");
				
			
		
	}
					
				
	System.out.append(s);
		}
    
# Proses yang sama diikuti untuk 16 bit key
System.out.println("\n\nmasukkan kunci 8 karakter :");
		String key = new Scanner(System.in).nextLine();
		String hexEquivalent2 = asciiToHex(key);
    
# nilai hexadecimal dari original string
hexEquivalent2 = hexEquivalent2.toUpperCase();
      System.out.println("Nilai Hexa: "+ hexEquivalent2);
      System.out.println("Nilai biner:");
		int keyBits[] = new int[64];
		for(int i=0 ; i < 16 ; i++) {
			String s = Integer.toBinaryString(Integer.parseInt(hexEquivalent2.charAt(i) + "", 16));
			while(s.length() < 4) {
				s = "0" + s;
			}
			for(int j=0 ; j < 4 ; j++) {
				keyBits[(4*i)+j] = Integer.parseInt(s.charAt(j) + "");
			}
			System.out.append(s);
			
		}
    
# permute(int[] inputBits, int[] keyBits, boolean isDecrypt) method digunakan disini. Agar proses enkripsi dan dekripsi dapat diselesaikan pada method yang sama dan mengurangi code.
System.out.println("\n\n====================================================================");
		System.out.println("ENCRYPTION");
		int outputBits[] = permute(inputBits, keyBits, false);
		System.out.println("\n\n====================================================================");
		System.out.println("DECRYPTION");
	
		permute(outputBits, keyBits, true);
		
	}
  
 private static int[] permute(int[] inputBits, int[] keyBits, boolean isDecrypt) 
# Langkah Initial permutation mengambil input bit dan mengubah urutannya ke array newBits
{
int newBits[] = new int[inputBits.length];
		for(int i=0 ; i < inputBits.length ; i++) {
			newBits[i] = inputBits[IP[i]-1];
		}
    
# 16 rounds akan dimulai pada langkah ini array L & R dibuat untuk menyimpan masing-masing subkey Left dan Right
int L[] = new int[32];
		int R[] = new int[32];
		int i;
		
		// Permuted Choice 1 sudah selesai pada tahap ini
		for(i=0 ; i < 28 ; i++) {
			C[i] = keyBits[PC1[i]-1];
		}
		for( ; i < 56 ; i++) {
			D[i-28] = keyBits[PC1[i]-1];
		}
    
# setelah proses PC1 baris pertama L dan R siap untuk digunakan dan looping dapat dimulai setelah L dan R sudah diinisialisasikan
System.arraycopy(newBits, 0, L, 0, 32);
		System.arraycopy(newBits, 32, R, 0, 32);
		System.out.print("\nL0 = ");
		displayBits(L);
		System.out.print("R0 = ");
		displayBits(R);
		for(int n=0 ; n < 16 ; n++) {
			System.out.println("\n-------------");
			System.out.println("Round " + (n+1) + ":");
      
# newr adalah setengah dari R baru yang dihasilkan oleh Fiestel function. Jika itu adalah enkripsi maka metode KS dipanggil untuk menghasilkan subkey jika tidak subkunci disimpan digunakan dalam urutan terbalik untuk proses dekripsi.
int newR[] = new int[0];
			if(isDecrypt) {
				newR = fiestel(R, subkey[15-n]);
				System.out.print("Round key = ");
				displayBits(subkey[15-n]);
			} else {
				newR = fiestel(R, KS(n, keyBits));
				System.out.print("Round key = ");
				displayBits(subkey[n]);
			}
      
# proses xor L dan R baru memberikan nilai L yang baru. L baru disimpan pada R dan R baru disimpan di L, untuk menukar R dan L pada round berikutnya
int newL[] = xor(L, newR);
			L = R;
			R = newL;
			System.out.print("L = ");
			displayBits(L);
			System.out.print("R = ");
			displayBits(R);
		}
    
# R dan L memiliki 2 bagian output sebelum menerapkan permutasi akhir proses ini disebut "Preoutput".
int output[] = new int[64];
		System.arraycopy(R, 0, output, 0, 32);
		System.arraycopy(L, 0, output, 32, 32);
		int finalOutput[] = new int[64];
    
# menerapkan tabel FP ke preoutput, kita dapat output akhir: Encryption => final output berupa ciphertext & Decryption => final output berupa plaintext
for(i=0 ; i < 64 ; i++) {
			finalOutput[i] = output[FP[i]-1];
		}
    
# sejak final output disimpan sebagai sebuah int bit array, lalu kita mengubahnya menjadi string hexadecimal:
String hex = new String();
		for(i=0 ; i < 16 ; i++) {
			String bin = new String();
			for(int j=0 ; j < 4 ; j++) {
				bin += finalOutput[(4*i)+j];
			}
			int decimal = Integer.parseInt(bin, 2);
			hex += Integer.toHexString(decimal);
		}
		String asciiEquivalent = hexToASCII(hex);
		if(isDecrypt) {
			System.out.println("\nNilai Hexa: "+hex.toUpperCase());
		
			 //nilai ASCII yang diperoleh dari nilai Hexadecimal
      
			System.out.println("Hasil Deskripsi: "+ asciiEquivalent);
		
		} else {
			System.out.println("\nNilai Hexa: "+hex.toUpperCase());
			
			System.out.println("Hasil Enkripsi: "+ asciiEquivalent);
		}
		
		
     
		return finalOutput;
			
	}
  
private static int[] KS(int round, int[] key){
# fungsi KS menghasilkan the round keys C1 dan D1 merupakan nilai baru dari C dan D yang mana akan diubah pada round ini.
int C1[] = new int[28];
int D1[] = new int[28];

# rotasi array digunakan untuk mengatur berapa banyak rotasi untuk diselesaikan
int rotationTimes = (int) rotations[round];

# method leftShift() digunakan untuk rotasi
C1 = leftShift(C, rotationTimes);
D1 = leftShift(D, rotationTimes);

# CnDn menyimpan kombinasi antara setengah dari C1 dan D1
int CnDn[] = new int[56];
		System.arraycopy(C1, 0, CnDn, 0, 28);
		System.arraycopy(D1, 0, CnDn, 28, 28);
    
# Kn menyimpan subkey, yang diubah oleh penerapan tabel PC2 ke CnDn
int Kn[] = new int[48];
		for(int i=0 ; i < Kn.length ; i++) {
			Kn[i] = CnDn[PC2[i]-1];
		}

# Method untuk menjalankan Fiestelfunction 32 bit pertama array R diekspansi menggunakan tabel E.
int expandedR[] = new int[48];
		for(int i=0 ; i < 48 ; i++) {
			expandedR[i] = R[E[i]-1];
		}
    
# Method untuk menampilkan bit int array sebagai sebuah string hexadecimal
for(int i=0 ; i < bits.length ; i+=4) {
			String output = new String();
			for(int j=0 ; j < 4 ; j++) {
				output += bits[i+j];
			}
			System.out.print(Integer.toHexString(Integer.parseInt(output, 2)));
		}
		System.out.println();
		
	}
  

