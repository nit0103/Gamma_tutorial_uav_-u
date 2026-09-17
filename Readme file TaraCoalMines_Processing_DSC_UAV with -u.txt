




# Note: do not put "space" in folders
cd /d/TaraCoalMines_Processing_DSC_UAV                                                               
/bin/ls -1d Data_Descending/dims*/TSX-1.*/TDX1* > TX_list                                                                                            
TX_SLC_preproc TX_list slc TX_SLC_preproc.log        # make gamma software compatible
mk_tab slc slc slc.par SLC_tab

base_calc SLC_tab slc/20260818T000659_TDX1_HH.slc.par TCM.berp_all itab_all 1 1 0.2

mk_mli_all SLC_tab mli_3_3 3 3 0 0.8 0.35
####################################################################################


cd DEM
dem_import TCM_UAV.tif TCM.dem TCM.dem_par 0 1 - - 0                                                ## file should be in same coordinate system
disdem_par.exe TCM.dem TCM.dem_par&
cd ..



________________________________________
mkdir geo

mk_geo_radcal2 mli_3_3/20260818T000659_TDX1_HH.mli mli_3_3/20260818T000659_TDX1_HH.mli.par DEM/TCM.dem DEM/TCM.dem_par DEM/TCM_seg.dem DEM/TCM_seg.dem_par geo TCM_seg 3.7e-5 0 3 -j -p -d -z

mk_geo_radcal2 mli_3_3/20260818T000659_TDX1_HH.mli mli_3_3/20260818T000659_TDX1_HH.mli.par DEM/TCM.dem DEM/TCM.dem_par DEM/TCM_seg.dem DEM/TCM_seg.dem_par geo TCM_seg 3.7e-5 1 3 -j -p -d -z

mk_geo_radcal2 mli_3_3/20260818T000659_TDX1_HH.mli mli_3_3/20260818T000659_TDX1_HH.mli.par DEM/TCM.dem DEM/TCM.dem_par DEM/TCM_seg.dem DEM/TCM_seg.dem_par geo TCM_seg 3.7e-5 2 3 -j -p -d -z

mk_geo_radcal2 mli_3_3/20260818T000659_TDX1_HH.mli mli_3_3/20260818T000659_TDX1_HH.mli.par DEM/TCM.dem DEM/TCM.dem_par DEM/TCM_seg.dem DEM/TCM_seg.dem_par geo TCM_seg 3.7e-5 3 3 -j -p -d -z


SLC_resamp_lt_all SLC_tab slc/20260818T000659_TDX1_HH.slc slc/20260818T000659_TDX1_HH.slc.par mli_3_3/20260818T000659_TDX1_HH.mli.par geo/TCM_seg_dem.rdc mli_3_3 rslc RSLC_tab 0 -b 128 -u

SLC_resamp_lt_all SLC_tab slc/20260818T000659_TDX1_HH.slc slc/20260818T000659_TDX1_HH.slc.par mli_3_3/20260818T000659_TDX1_HH.mli.par geo/TCM_seg_dem.rdc mli_3_3 rslc RSLC_tab 1 -b 128 -u

SLC_resamp_lt_all SLC_tab slc/20260818T000659_TDX1_HH.slc slc/20260818T000659_TDX1_HH.slc.par mli_3_3/20260818T000659_TDX1_HH.mli.par geo/TCM_seg_dem.rdc mli_3_3 rslc RSLC_tab 2 -b 128 -u

SLC_resamp_lt_all SLC_tab slc/20260818T000659_TDX1_HH.slc slc/20260818T000659_TDX1_HH.slc.par mli_3_3/20260818T000659_TDX1_HH.mli.par geo/TCM_seg_dem.rdc mli_3_3 rslc RSLC_tab 3 -b 128 -u

SLC_resamp_lt_all SLC_tab slc/20260818T000659_TDX1_HH.slc slc/20260818T000659_TDX1_HH.slc.par mli_3_3/20260818T000659_TDX1_HH.mli.par geo/TCM_seg_dem.rdc mli_3_3 rslc RSLC_tab 4 -b 128 -u


mk_mli_all RSLC_tab rmli_3_3 3 3 1 0.8 0.35 TCM_rmli.ave

___________________________________
mk_diff_2d RSLC_tab itab_all 0 geo/TCM_seg_dem.rdc - rmli_3_3/TCM_rmli.ave rmli_3_3 diff0_2d_all 3 3 3 1 0 -o -s 1.0 -e 0.3 -u
mk_adf_2d RSLC_tab itab_all rmli_3_3/TCM_rmli.ave diff0_2d_all 3 .4 80 20 -s 1.0 -e .3 -u
mk_tab diff0_2d_all adf.cc - diff0_2d_all/adf_cc.list
##############################################################

ave_image.exe diff0_2d_all/adf_cc.list 3157 diff0_2d_all/adf_cc.ave 1 - 1 1 0 -u

rasdt_pwr diff0_2d_all/adf_cc.ave - 3157 1 2764 1 1 0 1 0 cc.cm \
diff0_2d_all/adf_cc.ave.bmp 1.0 0.35 8

thres_data.exe diff0_2d_all/adf_cc.ave.bmp 3157 diff0_2d_all/mask_unw.bmp diff0_2d_all/adf_cc.ave 0.7 1.0 2 -u
disras diff0_2d_all/mask_unw.bmp&
mk_unw_2d RSLC_tab itab_all rmli_3_3/TCM_rmli.ave diff0_2d_all 0.0 .01 1 1 1 1 1518 1445 0 - - - - - -d diff_tab_all
_________________________

mk_dispmap_2d diff_tab_all rmli_3_3/TCM_rmli.ave \
rmli_3_3/20260818T000659_TDX1_HH.rmli.par \
disp0_2d_all 0.5 disp_tab_all 1 -m hls.cm -u


mkdir geocoded_products

mk_geo_data rmli_3_3/TCM_rmli.ave.par DEM/TCM_seg.dem_par geo/TCM_seg_1.map_to_rdc rmli_3_3/20260818T000659_TDX1_HH.rmli.bmp geocoded_products/TCM_rmli_geo.bmp 0 2 mk_geo_TCM_rmli.log
data2geotiff.exe DEM/TCM_seg.dem_par geocoded_products/TCM_rmli_geo.bmp 0 geocoded_products/TCM_rmli_geo.tif

mk_geo_data rmli_3_3/TCM_rmli.ave.par DEM/TCM_seg.dem_par geo/TCM_seg_1.map_to_rdc disp0_2d_all/20260818T000659_TDX1_HH_20260829T000659_TDX1_HH.adf.disp geocoded_products/disp_18082026_29082026_geo1 0 0 disp_18082026_29082026.log   
data2geotiff.exe DEM/TCM_seg.dem_par geocoded_products/disp_18082026_29082026_geo1 2 geocoded_products/disp_18082026_29082026.tif

mk_geo_data rmli_3_3/TCM_rmli.ave.par DEM/TCM_seg.dem_par geo/TCM_seg_1.map_to_rdc diff0_2d_all/20260818T000659_TDX1_HH_20260829T000659_TDX1_HH.adf.diff.bmp geocoded_products/adf_diff_18082026_29082026_geo.bmp 0 2 adf_diff_18082026_29082026.log   
data2geotiff.exe DEM/TCM_seg.dem_par geocoded_products/adf_diff_18082026_29082026_geo.bmp 0 geocoded_products/adf_diff_18082026_29082026.tif
