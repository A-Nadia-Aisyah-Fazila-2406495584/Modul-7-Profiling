# Tutorial Module 7

## Performance Testing

<details>

<summary>
<b>Run via GUI</b>
</summary>

### Test Plan 1 (/all-students)
![Test Plan 1 GUI image](/assets/images/gui_test_plan_1.png)

### Test Plan 2 (/all-student-name)
![Test Plan 2 GUI image](/assets/images/gui_test_plan_2.png)

### Test Plan 3 (/highest-gpa)
![Test Plan 3 GUI image](/assets/images/gui_test_plan_3.png)

</details>

<details>

<summary>
<b>Run via Command Line</b>
</summary>

### Command yang saya gunakan
```
./jmeter -n -t test_plan_1.jmx -l result_1.jtl
./jmeter -n -t test_plan_2.jmx -l result_2.jtl
./jmeter -n -t test_plan_3.jmx -l result_3.jtl
```

### Test Plan 1 (/all-students)
![Test Plan 1 CL image](/assets/images/cl_test_plan_1.png)

### Test Plan 2 (/all-student-name)
![Test Plan 2 CL image](/assets/images/cl_test_plan_2.png)

### Test Plan 3 (/highest-gpa)
![Test Plan 3 CL image](/assets/images/cl_test_plan_3.png)

</details>

### Summary 
- Test Plan 1 Memiliki sample time yang jauh lebih tinggi dibandingkan dengan Test Plan 2 dan Test Plan 3 karena endpoint ini mengambil seluruh data student beserta dengan courses nya secara lengkap. Hal ini menyebabkan (N+1) query problem, dimana untuk setiap student (20.000 data), aplikasi mengirimkan query tambahan ke database untuk ambil data courses nya masing-masing. Sehingga, total query yang dieksekusi bisa sampai puluhan ribu.
- Test Plan 2 Lebih cepat dari Test Plan 1 karena hanya mengambil nama student saja, sehingga data yang diproses dan dikembalikan jauh lebih sedikit.
- Test Plan 3 Jadi yang tercepat karena hanya mengembalikan satu data saja yaitu student dengan GPA tertinggi, sehingga komputasi dan transfer datanya minimal.

## Profiling
<details>

<summary>
<b>Endpoint /all-student</b>
</summary>

- JMETER 
  - before: ~42049 ms
    ![Test Plan 1 CL image](/assets/images/cl_test_plan_1.png)
  - after: ~237 ms
    ![Test Plan 1 CL After_image](/assets/images/cl_test_plan_1_after.png)
  - summary:
    Sebelum optimasi, rata-rata response time endpoint /all-student di JMeter adalah sekitar 42049 ms (42 detik). Setelah optimasi, turun drastis menjadi sekitar 237 ms. Selisihnya adalah sekitar 41812 ms dengan improvement sebesar ~99.4%.

- Intellij Profiler (CPU Time `getAllStudentsWithCourses`)
  - before: 9576 ms
    ![Profiling_all-student_Before_image](/assets/images/profiling_all-student.png)
  - after: 270 ms
    ![Profiling_all-student_After_image](/assets/images/profiling_all-student_after.png)
  - comparison: 9306 ms
    ![Profiling_all-student_After_image](/assets/images/profiling_all-student_comparison.png)
  - summary:
    Dari hasil profiling, CPU time method `getAllStudentsWithCourses` sebelum optimisasi adalah 9576 ms, dan setelah optimasi turun menjadi 270 ms. Selisihnya adalah 9,306 ms dengan improvement sebesar ~97.2%, sesuai dengan yang terlihat di comparison view.

- Masalahnya ada di method `getAllStudentsWithCourses` yang punya N+1 query problem. Kode lama itu query semua student dulu, lalu untuk setiap student dia query lagi ke tabel `student_courses` satu per satu. Dengan 20.000 student berarti ada 20.001 query yang dikirim ke database setiap kali endpointnya diakses, jadinya lambat hingga ~42 detik. Solusinya adalah mengganti dengan satu query pakai `JOIN FETCH` di repository supaya semua data student dan courses nya langsung diambil dalam 1 query, tanpa harus bolak balik ke database berkali-kali. Setelah itu, method di service nya tinggal panggil method repository yang baru tanpa perlu logic looping lagi.

</details>

<details>

<summary>
<b>Endpoint /all-student-name</b>
</summary>

- JMETER
  - before: ~1033 ms
    ![Test Plan 2 CL image](/assets/images/cl_test_plan_2.png)
  - after: ~25 ms
    ![Test Plan 2 CL After_image](/assets/images/cl_test_plan_2_after.png)
  - summary:
    Sebelum optimasi, rata-rata response time endpoint /all-student-name di JMeter adalah sekitar 1033 ms. Setelah optimasi, turun drastis menjadi sekitar 25 ms. Selisihnya adalah sekitar 1008 ms dengan improvement sebesar ~97.6%.

- Intellij Profiler (CPU Time `joinStudentNames`)
  - before: 422 ms
    ![Profiling_all-student-name_Before_image](/assets/images/profiling_all-student-name.png)
  - after: 50 ms
    ![Profiling_all-student-name_After_image](/assets/images/profiling_all-student-name_after.png)
  - comparison: 372 ms
    ![Profiling_all-student-name_Comparison_image](/assets/images/profiling_all-student-name_comparison.png)
  - summary:
    Dari hasil profiling, CPU time method `joinStudentNames` sebelum optimasi adalah 422 ms, dan setelah optimasi turun menjadi 50 ms. Selisihnya adalah 372 ms dengan improvement sebesar ~88.2%, sesuai dengan yang terlihat di comparison view.

- Masalahnya ada di method `joinStudentNames` yang melakukan string concatenation menggunakan `+=` di dalam loop. Setiap operasi `+=` pada String di Java itu membuat object String baru di memory, jadi dengan 20.000 student ada 20.000 object String yang dibuat. Selain itu, kode lama juga mengambil seluruh object Student padahal yang dibutuhkan hanya nama saja. Solusinya adalah query langsung ambil kolom `name` saja dari database pakai `@Query`, lalu hasilnya digabung pakai `String.join()` yang lebih efisien karena tidak membuat object String baru berkali-kali. Setelah itu, method di service nya juga tinggal panggil method repository yang baru tanpa perlu logic looping lagi.

</details>

<details>

<summary>
<b>Endpoint /highest-gpa</b>
</summary>

- JMETER
  - before: ~67 ms
    ![Test Plan 3 CL image](/assets/images/cl_test_plan_3.png)
  - after: ~10 ms
    ![Test Plan 3 CL After_image](/assets/images/cl_test_plan_3_after.png)
  - summary:
    Sebelum optimasi, rata-rata response time endpoint /highest-gpa di JMeter adalah sekitar 67 ms. Setelah optimasi, turun menjadi sekitar 10 ms. Selisihnya adalah sekitar 57 ms dengan improvement sebesar ~85%.

- Intellij Profiler (CPU Time `findStudentWithHighestGpa`)
  - before: 130 ms
    ![Profiling_highest-gpa_Before_image](/assets/images/profiling_highest-gpa.png)
  - after: 50 ms
    ![Profiling_highest-gpa_After_image](/assets/images/profiling_highest-gpa_after.png)
  - comparison: 80 ms
    ![Profiling_highest-gpa_Comparison_image](/assets/images/profiling_highest-gpa_comparison.png)
  - summary:
    Dari hasil profiling, CPU time method `findStudentWithHighestGpa` sebelum optimasi adalah 130 ms, dan setelah optimasi turun menjadi 50 ms. Selisihnya adalah 80 ms dengan improvement sebesar ~61.5%, sesuai dengan yang terlihat di comparison view.

- Masalahnya ada di method `findStudentWithHighestGpa` yang ambil semua 20.000 data student dari database hanya untuk mencari 1 student dengan GPA tertinggi. Logika pencariannya dilakukan di Java dengan loop satu per satu, padahal database jauh lebih efisien untuk urusan sorting dan filtering. Solusinya adalah mendelegasikan pencarian langsung ke database menggunakan konvensi JPA `findTopByOrderByGpaDesc`, yang artinya database langsung mengurutkan berdasarkan GPA secara descending dan mengembalikan hanya 1 data teratas. Setelah itu, method di service nya juga tinggal panggil method repository yang baru tanpa perlu logic looping lagi.

</details>

## Reflection
**1. What is the difference between the approach of performance testing with JMeter and profiling with IntelliJ Profiler in the context of optimizing application performance?**
JMeter itu fokusnya di sisi luar aplikasi, dia simulate banyak user yang hit endpoint secara bersamaan, lalu ukur berapa lama response timenya. Jadi JMeter ngasih tau kalo suatu endpoint itu lambat, tapi tidak tau kenapa lambatnya. Nah, IntelliJ Profiler itu yang ngasih jawaban kenapa nya, dia masuk ke dalam aplikasi dan ngukur method mana yang paling banyak makan CPU time, memory, dll. Jadi keduanya itu saling melengkapi, JMeter untuk deteksi masalah dari luar dan Profiler untuk diagnosa dari dalam.

**2. How does the profiling process help you in identifying and understanding the weak points in your application?**
Profiling sangat membantu karena dia bisa tunjukin secara visual method mana yang paling lama dieksekusi. Tanpa profiling, harus nebak-nebak dulu bagian kode mana yang bermasalah. Dengan profiling, langsung keliatan misalnya `getAllStudentsWithCourses` itu CPU time nya jauh lebih tinggi dari method lain, jadi kita tau persis harus optimasi di method mana.

**3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify bottlenecks in your application code?**
Ya sangat efektif karena memudahkan untuk identifikasi bottleneck karena tampilannya visual dan langsung bisa di drill down sampai ke level method. Fitur comparison view nya juga berguna untuk ukur apakah optimasi yang dilakukan benar-benar berdampak.

**4. What are the main challenges you face when conducting performance testing and profiling, and how do you overcome these challenges?**
Tantangan utamanya ada beberapa, diantaranya adalah:
  - Hasil profiling bisa tidak konsisten kalau dijalankan di run pertama karena JIT compiler belum warm up, jadi harus run beberapa kali dulu sebelum ambil data yang valid. 
  - Agak tricky untuk memastikan profiler merekam request yang tepat, karena kalau ada request lain (ch adalah seeding data) yang masuk duluan, hasilnya bisa tercampur. Cara mengatasinya adalah dengan memastikan hanya akses endpoint yang mau diprofiling selama recording berlangsung.

**5. What are the main benefits you gain from using IntelliJ Profiler for profiling your application code?**
Manfaat utamanya adalah bisa langsung tau method mana yang jadi bottleneck tanpa harus baca kode satu per satu. Selain itu, fitur comparison view memudahkan validasi bahwa optimasi yang dilakukan benar-benar improve performance secara terukur. Karena langsung terintegrasi di IntelliJ, workflow nya juga lebih smooth dibanding harus pakai tool yang terpisah.

**6. How do you handle situations where the results from profiling with IntelliJ Profiler are not entirely consistent with findings from performance testing using JMeter?**
Jika ada inkonsistensi, biasanya lihat dulu konteksnya masing-masing. JMeter mengukur end-to-end termasuk network latency, serialisasi response, dll. Sedangkan, Profiler hanya ukur CPU time di dalam kode Java nya. Jadi wajar jika angkanya berbeda. Yang penting adalah trennya konsisten, jika JMeter bilang ada improvement dan Profiler juga konfirmasi ada penurunan CPU time, artinya optimasinya berhasil. Jika benar-benar kontradiktif, maka perlu investigasi lebih lanjut, misalnya dengan cek apakah ada bottleneck lain di luar kode seperti koneksi database atau network.

**7. What strategies do you implement in optimizing application code after analyzing results from performance testing and profiling? How do you ensure the changes you make do not affect the application's functionality?**
Strateginya adalah fokus dulu ke bottleneck yang paling besar dampaknya. Dari hasil profiling, identifikasi method dengan CPU time tertinggi, lalu analisis penyebabnya, apakah N+1 query, inefficient loop, atau unnecessary data fetching. Setelah ketemu masalahnya, baru refactor dengan solusi yang tepat seperti JOIN FETCH atau mendelegasikan logika ke database. Untuk memastikan fungsionalitasnya tidak berubah, selalu test endpoint setelah optimasi untuk verifikasi response nya masih sama, dan jalankan ulang JMeter untuk konfirmasi ada improvement performa.

