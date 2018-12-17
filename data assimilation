subroutine da_read_obs_netcdf4ahi_geocat (iv, infile_tb, infile_clp)
   !--------------------------------------------------------
   !  Purpose: read in GEOCAT AHI Level-1 and Level-2 data in NETCDF4 format
   !           and form innovation structure
   !
   !   METHOD: use F90 sequantial data structure to avoid read the file twice
   !            1. read file radiance data in sequential data structure
   !            2. do gross QC check
   !            3. assign sequential data structure to innovation structure
   !                  and deallocate sequential data structure
   !
   !  HISTORY: 2016/10/22 - Creation         Yuanbing Wang, NUIST/CAS, NCAR/NESL/MMM/DAS
   !  Update : 2018/1/25  - Lideqin , add allsky variables, missing value
   !  To be devoloped: 1.time information; 2.dimension sequence
   !------------------------------------------------------------------------------

   use netcdf
   implicit none

   character(len=*), intent(in)    :: infile_tb, infile_clp
   type(iv_type),    intent(inout) :: iv
! fixed parameter values
   integer,parameter::time_dims=6       ! Time dimension
   integer,parameter::nfile_max = 8     ! each netcdf file contains
   real,parameter::add_offset_tb1=285.0
   real,parameter::add_offset_tb2=235.0
   real,parameter::add_offset_tb3=260.0
   real,parameter::add_offset_tb4=240.0
   real,parameter::add_offset_saz=90.0
   real, parameter :: add_offset_suz=90.0
   real,parameter::scale_factor_tb1=0.00350962858973968
   real,parameter::scale_factor_tb2=0.00198370311593982
   real,parameter::scale_factor_tb3=0.00274666585283975
   real,parameter::scale_factor_tb4=0.00213629566331980
   real,parameter::scale_factor_lat=0.00274666585283975
   real,parameter::scale_factor_lon=0.00549333170567949
   real,parameter::scale_factor_saz=0.00274666585283975
   real, parameter :: scale_factor_suz=0.00274666585283975
! interface variable
   integer iret, rcode, ncid                      ! return status
! array data
   real(4), allocatable  :: vlatitude(:,:)  !  value for latitude
   real(4), allocatable  :: vlongitude(:,:) !  value for longitude
   real(4), allocatable  :: tbb(:,:,:)  ! tb for band 7-16
   real(4), allocatable  :: sat_zenith(:,:)
   real(4), allocatable  :: sun_zenith(:,:)
   byte, allocatable     :: sur_type(:,:)
   byte, allocatable     :: cloud_mask(:,:)
   byte, allocatable     :: land_mask(:,:)
   real(4), allocatable  :: SD_ob(:,:)

   real(r_kind),parameter  :: tbmin  = 50._r_kind
   real(r_kind),parameter  :: tbmax  = 550._r_kind

   real(kind=8)                   :: obs_time
   type (datalink_type),pointer   :: head, p, current, prev
   type(info_type)                :: info
   type(model_loc_type)           :: loc

   integer(i_kind)    :: idate5(6)
   character(len=80)  :: filename,str_tmp
