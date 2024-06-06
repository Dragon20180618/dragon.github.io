date: 2024-06-06 15:53:08

tags: Fortran dynamic DLL LoadLibrary

+ Source code of DLL

    ```fortran
    subroutine test(a, n)
        implicit none
        !dec$ attributes dllexport::test
        integer::n
        integer::a(n)
        integer::i
        do i = 1, n
            a(i) = i*i
        end do
    end subroutine test
    ```

* Source code of executable file

  ```fortran
  use iso_c_binding
  implicit none
  interface
      function LoadLibrary(name) bind(c, name='LoadLibraryA')
          import :: c_ptr, c_char
          character(kind=c_char), intent(in) :: name(*)
          type(c_ptr) :: LoadLibrary
      end function LoadLibrary
  
      function GetProcAddress(hModule, lpProcName) bind(c, name='GetProcAddress')
          import :: c_ptr, c_char, c_funptr
          type(c_ptr), value :: hModule
          character(kind=c_char), intent(in) :: lpProcName(*)
          type(c_funptr) :: GetProcAddress
      end function GetProcAddress
  
      subroutine FreeLibrary(hModule) bind(c, name='FreeLibrary')
          import :: c_ptr
          type(c_ptr), value :: hModule
      end subroutine FreeLibrary
  end interface
  
  interface
      subroutine test_interface(a,n)
          integer::n
          integer::a(n)
      end subroutine test_interface
  end interface
  
  integer, parameter::n = 5
  integer::a(n)
  
  type(c_ptr) :: funcdll
  type(c_funptr) :: funcptr
  procedure(test_interface), pointer :: test => null()
  
  funcdll = LoadLibrary("./func.dll"//c_null_char)
  if (.not. C_ASSOCIATED(funcdll)) then
      print *, "Failed to load DLL."
  end if
  funcptr = GetProcAddress(funcdll, "TEST"//c_null_char)
  
  if (.not. C_ASSOCIATED(funcptr)) then
      print *, "Failed to load func."
  end if
  
  call c_f_procpointer(funcptr,test)
  
  call test(a,n)
  print *,a
  
  call FreeLibrary(funcdll)
  end
  
  ```
  

+ compile commands

    + generate func.dll

	  `ifx /DLL /o func.dll func.f90`

    + generate exe
    
      `ifx main.f90`

+ console output

  ​           1           4           9          16          25
