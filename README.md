# CLAW language definition

This file contains the 1st iteration for the directive language of the CLAW
project


<!---

  LOOP TRANSFORMATION SECTION

--->

## Transformation on loops

#### Loop interchange
##### Directive defintion
<!--- TODO define a notion of dependency --->
<!--- TODO define a notion of depth --->
```fortran
!$claw loop-interchange
```

The loop-interchange directive allows two loops to swap their place.

##### Example 1
###### Original code
```fortran
!$claw loop-interchange
DO i=1, iend
    DO k=1, kend
      ! loop body here
    ENDDO
  ENDDO
ENDDO
```

###### Transformed code
```fortran
! CLAW transformation (loop-interchange i < -- > k)
DO k=1, kend
  DO i=1, iend
    ! loop body here
  ENDDO
ENDDO
```
##### Example 2
###### Original code
```fortran
!$claw loop-interchange [new-order(k,i,j)]
DO i=1, iend     ! loop at depth 0
  DO j=1, jend   ! loop at depth 1
    DO k=1, kend ! loop at depth 2
      ! loop body here
    ENDDO
  ENDDO
ENDDO
```

###### Transformed code
```fortran
! CLAW transformation (loop-interchange new-order(k,i,j))
DO k=1, kend       ! loop at depth 2
  DO i=1, iend     ! loop at depth 0
    DO j=1, jend   ! loop at depth 1
      ! loop body here
    ENDDO
  ENDDO
ENDDO
```



#### Loop fusion
###### Directive defintion
```fortran
!$claw loop-fusion [group(*group_id*:*pos*)]
```

The loop-fuson directive allows to fusion 2 to N loops in a single one. If no
group option is given, all the loops decorated with the directive in the same
block will be fusioned together as a single group.

If the *group* option is given, the loops are fusioned within the given group
according to their position.

All the loop within a group must share the same range.

##### Example 1 (without *group* option)
###### Original code
```fortran
DO k=1, iend
  !$claw loop-fusion
  DO i=1, iend
    ! loop #1 body here
  ENDDO

  !$claw loop-fusion
  DO i=1, iend
    ! loop #2 body here
  ENDDO
ENDDO
```

###### Transformed code
```fortran
DO k=1, iend
  ! CLAW transformation (loop-fusion same block group)
  DO i=1, iend
    ! loop #1 body here
    ! loop #2 body here
  ENDDO
ENDDO
```


##### Example 2 (with *group* option)
###### Original code
```fortran
DO k=1, iend
  !$claw loop-fusion group(g1:1)
  DO i=1, iend
    ! loop #1 body here
  ENDDO

  !$claw loop-fusion group(g1:2)
  DO i=1, iend
    ! loop #2 body here
  ENDDO

  !$claw loop-fusion group(g2:1)
  DO i=1, jend
    ! loop #3 body here
  ENDDO

  !$claw loop-fusion group(g2:1)
  DO i=1, jend
    ! loop #4 body here
  ENDDO
ENDDO
```

###### Transformed code
```fortran
DO k=1, iend
  ! CLAW transformation (loop-fusion group g1)
  DO i=1, iend
    ! loop #1 body here
    ! loop #2 body here
  ENDDO

  ! CLAW tranformation (loop-fusion group g2)
  DO i=1, jend
    ! loop #3 body here
    ! loop #4 body here
  ENDDO
ENDDO
```


<!---

  VARIABLE TRANSFORMATION SECTION

--->

## Transformation on variable
<!--- TODO all reflexion and definition --->
#### Demotion
###### Directive defintion
```fortran
!$claw demote(variable_list) dim(dimension_from,dimension_to)
```

###### Original code
```fortran
SUBROUTINE xyz(value1, value2)
  REAL, INTENT (IN) :: value2(x:y), value2(x:y)

  DO i = 0, iend
    ! some computation with value1(i) here
  END DO
END SUBROUTINE xyz

!$claw demote(value1, value2) (1d, 0d)
CALL xyz(value1, value2)
```

###### Transformed code
```fortran
!CLAW transformation (demotion of variable (value1, value2) (1d,0d))
SUBROUTINE xyz_claw(value1_claw, value2_claw)
  REAL, INTENT (IN) :: value1_claw, value2_claw
  ! some computation with value here
END SUBROUTINE

!CLAW transformation (demotion of variable (value1, value2) (1d,0d))
DO i = 0, iend
  CALL xyz_claw(value1_claw(i), value2_claw(i))
END DO
```
