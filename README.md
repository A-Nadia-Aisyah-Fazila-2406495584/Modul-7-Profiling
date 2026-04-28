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
  - before: ~42000 ms
    ![Test Plan 1 CL image](/assets/images/cl_test_plan_1.png)
  - after: ~230 ms
    ![Test Plan 1 CL After_image](/assets/images/cl_test_plan_1_after.png)
  - summary:
    Sebelum optimasi, rata-rata response time endpoint /all-student di JMeter adalah sekitar 42000 ms (42 detik). Setelah optimasi, turun drastis menjadi sekitar 230 ms. Selisihnya adalah sekitar 41770 ms dengan improvement sebesar ~99%.

- Intellij Profiler (CPU Time `getAllStudentsWithCourses`)
  - before: 9576 ms
    ![Profiling_all-student_Before_image](/assets/images/profiling_all-student.png)
  - after: 270 ms
    ![Profiling_all-student_After_image](/assets/images/profiling_all-student_after.png)
  - comparison: 9306 ms
    ![Profiling_all-student_After_image](/assets/images/profiling_all-student_comparison.png)
  - summary:
    Dari hasil profiling, CPU time method `getAllStudentsWithCourses` sebelum optimisasi adalah 9576 ms, dan setelah optimasi turun menjadi 270 ms. Selisihnya adalah 9,306 ms dengan improvement sebesar ~97.2%, sesuai dengan yang terlihat di comparison view.

- Masalahnya ada di method `getAllStudentsWithCourses` yang punya N+1 query problem. Kode lama itu query semua student dulu, lalu untuk setiap student dia query lagi ke tabel `student_courses` satu per satu. Dengan 20.000 student berarti ada 20.001 query yang dikirim ke database setiap kali endpointnya diakses, jadinya lambat hingga ~42 detik. Solusinya adalah mengganti dengan satu query pakai `JOIN FETCH` di repository supaya semua data student dan courses nya langsung diambil dalam 1 query, tanpa harus bolak-balik ke database berkali-kali. Setelah itu, method di service nya tinggal panggil method repository yang baru tanpa perlu logic looping lagi.

</details>

<details>

<summary>
<b>Endpoint /all-student-name</b>
</summary>

- JMETER
  - before: ~1000 ms
    ![Test Plan 2 CL image](/assets/images/cl_test_plan_2.png)
  - after: ~25 ms
    ![Test Plan 2 CL After_image](/assets/images/cl_test_plan_2_after.png)
  - summary:
    Sebelum optimasi, rata-rata response time endpoint /all-student-name di JMeter adalah sekitar 1000 ms. Setelah optimasi, turun drastis menjadi sekitar 25 ms. Selisihnya adalah sekitar 975 ms dengan improvement sebesar ~97%.

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

## Reflection
**1. What is the difference between the approach of performance testing with JMeter and profiling with IntelliJ Profiler in the context of optimizing application performance?**

**2. How does the profiling process help you in identifying and understanding the weak points in your application?**

**3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify bottlenecks in your application code?**

**4. What are the main challenges you face when conducting performance testing and profiling, and how do you overcome these challenges?**

**5. What are the main benefits you gain from using IntelliJ Profiler for profiling your application code?**

**6. How do you handle situations where the results from profiling with IntelliJ Profiler are not entirely consistent with findings from performance testing using JMeter?**

**7. What strategies do you implement in optimizing application code after analyzing results from performance testing and profiling? How do you ensure the changes you make do not affect the application's functionality?**


