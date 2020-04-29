# JDK8相关

 - [分组](#分组)
  
 ### 分组
     /*
     1、社保大库
     */ EmployeeInsurance employeeCommonInsuranceVo = new EmployeeInsurance();
     employeeCommonInsuranceVo.setInsuranceId(Constants.SOCIAL_SECURITY_BIG);
             List<Long> commonWorkOrderIds = Optional.of(employeeCommonInsuranceVo)     
                     .map(employeeInsuranceMapper::getEmployeeInsuranceSameStorageInOrDe)    	    
                     .filter(li -> li.size() > 1)    	    
                     .map(li -> li.stream().flatMap(empVo -> {     	    
                         List<WorkOrderSameStorageResultVo> workOrders = Optional.ofNullable(empVo.getId()).
                         map(Long::valueOf)
                                 .map(WorkOrderSameStorageVo::new).map(workOrderMapper::
                                 getWorkOrderByEmployeeInsuranceId)
                                 .orElse(emptyList());			
                         return workOrders.stream()
                                 .filter(wo -> Byte.valueOf("1").equals(wo.getType()) || Byte.valueOf("2").
                                 equals(wo.getType()) || Byte.valueOf("3").equals(wo.getType()))
                                 .collect(groupingBy(this::fetchGroupKey))
                                 .entrySet().stream().filter(wo -> wo.getValue().stream().count() > 1)
                                 .flatMap(wo -> wo.getValue().stream().map(WorkOrderSameStorageResultVo::
                                 getId));
                     }).collect(Collectors.toList())).orElse(emptyList());
             if (commonWorkOrderIds.size()==2){
                 logger.error("同库增减执行任务失败");
                 returnData.setMsg("同库增减执行任务失败");
                 returnData.setRetCode(CommonConstants.RETURN_CODE_FAIL);
                 return returnData;
             }
     /*
              * 2、公积金大库
              * */
             EmployeeInsurance employeeCommonFundInsuranceVo = new EmployeeInsurance();
             employeeCommonFundInsuranceVo.setInsuranceId(Integer.valueOf(Constants.HOUSING_FOUND_BIG));
             List<EmployeeInsuranceSameVo> employeeCommonFundInsurances = employeeInsuranceMapper.
             getEmployeeInsuranceSameStorageInOrDe(employeeCommonFundInsuranceVo);
             logger.info("查询公积金大库中有两条及两条以上服务单的人数:{}", employeeCommonFundInsurances.size());
             if(employeeCommonFundInsurances.size()>=1){
                 employeeCommonFundInsurances.forEach(employeeCommonFundInsurance->{
                     WorkOrderSameStorageVo workOrderSameStorageVo =  new WorkOrderSameStorageVo
                     (Long.valueOf((employeeCommonFundInsurance.getId())));
                     List<WorkOrderSameStorageResultVo> workOrders = workOrderMapper.
                     getWorkOrderByEmployeeInsuranceId(workOrderSameStorageVo);
                     logger.info("workOrders.size():{}",workOrders.size());
                     Map<String, List<WorkOrderSameStorageResultVo>> WorkOrderSameStorageResults = 
                     workOrders.stream().filter(workOrder -> Byte.valueOf("1").equals(workOrder.getType()) 
                     || Byte.valueOf("3").equals(workOrder.getType()))
                             .collect(groupingBy(wo -> wo.getUserId() + new SimpleDateFormat("yyyy-MM")
                             .format(wo.getEffectTime()) + wo.getVendorId() + wo.getLocationId()+wo.
                             getEmployeeInsuranceId()));
                     List<WorkOrderSameStorageResultVo> commonFundWorkOrders = WorkOrderSameStorageResults.
                     entrySet().stream().flatMap(s -> s.getValue().stream()).collect(toList());
                     if(commonFundWorkOrders.size()>0){
                         commonFundWorkOrders.forEach(wo->{
                             if("0".equals(wo.getTagInfo())){
                                 WorkOrderWithBLOBs wob = new WorkOrderWithBLOBs();
                                 wob.setId(wo.getId());
                                 wob.setTagInfo("sameStorage");
                                 wob.setPendingDate(DateUtils.parseDate(String.valueOf(DateUtils.
                                 getCurrentTimeBySecond()), "yyyy-MM-dd"));
                                 workOrderMapper.updateByPrimaryKeySelective(wob);
                             }
                             if(!"0".equals(wo.getTagInfo()) && !wo.getTagInfo().contains("sameStorage")){
                                 WorkOrderWithBLOBs wob = new WorkOrderWithBLOBs();
                                 wob.setId(wo.getId());
                                 wob.setTagInfo(wo.getTagInfo()+","+"sameStorage");
                                 wob.setUpdateTime(DateUtils.getCurrentTimeBySecond());
                                 workOrderMapper.updateByPrimaryKeySelective(wob);
                             }
                        });
     
                     }
                 });
             }
     //3、取对象的集合进行操作
     List collect = commonFundWorkOrders.stream().map(WorkOrderSameStorageResultVo::getType).collect(toList());
     boolean commonFund = collect.contains("3") && collect.contains("2") || collect.contains("1");
 
     
> reubenwang@foxmail.com
> 没事别找我，找我也不在！--我很忙🦆