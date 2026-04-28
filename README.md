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

## Reflection
**1. What is the difference between the approach of performance testing with JMeter and profiling with IntelliJ Profiler in the context of optimizing application performance?**

**2. How does the profiling process help you in identifying and understanding the weak points in your application?**

**3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify bottlenecks in your application code?**

**4. What are the main challenges you face when conducting performance testing and profiling, and how do you overcome these challenges?**

**5. What are the main benefits you gain from using IntelliJ Profiler for profiling your application code?**

**6. How do you handle situations where the results from profiling with IntelliJ Profiler are not entirely consistent with findings from performance testing using JMeter?**

**7. What strategies do you implement in optimizing application code after analyzing results from performance testing and profiling? How do you ensure the changes you make do not affect the application's functionality?**


