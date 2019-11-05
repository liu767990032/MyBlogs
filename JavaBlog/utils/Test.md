package com.unicom.bi.appservice.search.manager.impl;

import java.text.SimpleDateFormat;
import java.util.List;
import java.util.Map;

import com.unicom.bi.appservice.scoreboard.dao.ScoreboardDao;
import net.sf.json.JSONArray;
import net.sf.json.JSONObject;

import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;
import org.jeecgframework.core.util.StringUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.alibaba.fastjson.JSON;
import com.unicom.bi.appservice.community.dao.CommunityDao;
import com.unicom.bi.appservice.community.service.CommunityService;
import com.unicom.bi.appservice.entity.CommunityComment;
import com.unicom.bi.appservice.search.dao.SearchDao;
import com.unicom.bi.appservice.search.manager.SearchManager;
import com.unicom.bi.appservice.service.tempdata.MB_39_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_40_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_42_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_43_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_44_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_45_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_46_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.MB_47_TempServiceImpl;
import com.unicom.bi.appservice.service.tempdata.TempUtil;
import com.unicom.bi.appservice.util.ConstantMe;
import com.unicom.bi.appservice.util.ExceptionUtil;
import com.unicom.bi.system.util.SystemParamUtil;
import com.unicom.bi.system.util.UserTokenUtil;

@Service
public class SearchManagerImpl implements SearchManager{
	@Autowired
	private CommunityDao communityDao;
	@Autowired
	private CommunityService communityService;
	@Autowired
	private SearchDao dao;
    @Autowired
    private TempUtil tempUtil;
    @Autowired
    private ScoreboardDao scoreboardDao;
	private static final Logger logger = Logger.getLogger(SearchManagerImpl.class);
	SimpleDateFormat sdf2 = new SimpleDateFormat("yyyyMMdd");
	/*
	 * 注入系统参数工具类
	 */
	@Autowired
	private SystemParamUtil systemParamUtil;
	/**
	 * SearchManager板块
	 * 获取热门报表、指标
	 * @return
	 */
	@SuppressWarnings("finally")
	public String getHotReportTarget(String userId, String tokenId) {
		JSONObject jObject = new JSONObject();
		if (!tempUtil.validUserToken(userId, tokenId)) {
			jObject.put("code", "102");
			jObject.put("message", UserTokenUtil.ERROR_MESSAGE);
			return jObject.toString();
		}
		try {
			JSONArray reportArray = new JSONArray();
			JSONArray targetArray = new JSONArray();
			List<Map<String, Object>> reportList = dao.getHotReportTarget("report", userId, "");
			List<Map<String, Object>> targetList = dao.getHotReportTarget("target", userId, "");
			if(reportList == null || reportList.size() == 0){
				reportList = dao.getHotReportTarget("report", userId, ConstantMe.HOT_REPORTTARGET_DEFAULT);
			}
			if (reportList != null && reportList.size() > 0) {
				for (Map<String, Object> teMap : reportList) {
					JSONObject joTemp = new JSONObject();
					String reportId = (String)teMap.get("REPORT_ID");
					joTemp.put("report_alias", teMap.get("ALIAS"));
					joTemp.put("report_id", teMap.get("REPORT_ID"));
					//APP排行调用的方法和别的方法不同，所以此处要特殊处理
					if("5000089901".equals(reportId) || "5000089902".equals(reportId)){
						joTemp.put("type_flag", "h");
					}else{
						joTemp.put("type_flag", teMap.get("TYPE"));
					}
					joTemp.put("type", teMap.get("TYPE"));
					reportArray.add(joTemp);
				}
			}
			if (targetList == null || targetList.size() == 0) {
				targetList = dao.getHotReportTarget("target", userId, ConstantMe.HOT_REPORTTARGET_DEFAULT);
			}
			if (targetList != null && targetList.size() > 0) {
				for (Map<String, Object> teMap : targetList) {
					JSONObject joTemp = new JSONObject();
					joTemp.put("target_alias", teMap.get("ALIAS"));
					joTemp.put("report_id", teMap.get("REPORT_ID"));
					joTemp.put("target_id", teMap.get("TARGET_ID"));
					joTemp.put("type", teMap.get("TYPE"));
					joTemp.put("type_flag", teMap.get("TYPE"));
					targetArray.add(joTemp);
				}
			}
			jObject.put("code", 1);
			jObject.put("hot_report", reportArray);
			jObject.put("hot_target", targetArray);
			jObject.put("message", "成功");
		} catch (Exception e) {
			jObject.put("code", "0");
			jObject.put("message", "接口调用失败！失败原因：" + e.getMessage());
		} finally {
			return jObject.toString();
		}
	}

	/**
	 * SearchManager板块
	 */
	@SuppressWarnings("finally")
	public String saveSearchReportTargetInfo(String userId, String reportId, String targetId, String areaId,
			String provId, String logType, String tokenId) {
		JSONObject jsonObject = new JSONObject();
		if (!tempUtil.validUserToken(userId, tokenId)) {
			jsonObject.put("code", "102");
			jsonObject.put("message", UserTokenUtil.ERROR_MESSAGE);
			return jsonObject.toString();
		}
		try {
			String monthId = "";
			String yunying_reports = systemParamUtil.getParamValue("yunying_reports");
			String day_report = systemParamUtil.getParamValue("day_report");
			// 获取报表的最新账期
			if (day_report.indexOf(reportId) > -1) {
				monthId = dao.getDayIdByReportId(reportId);
			} else {
				String paramValue = dao.getDayIdByReportId(reportId);
				if (StringUtil.isNotEmpty(paramValue)) {
					// 先取系统配置中最新的账期
					monthId = paramValue.split("#")[1].replaceAll("-", "");
				} else {
					if ("ex0260101,ex0260201,ex0260301,ex0260401,ex0260501,ex0260601".indexOf(reportId) > -1) {
						monthId = dao.getLastestMonthBf();
					} else {
						monthId = dao.getLastestMonth();
					}
				}
			}
			dao.saveSearchReportTargetInfo(userId, reportId, targetId, areaId, provId, logType, monthId);
			jsonObject.put("code", "1");
			jsonObject.put("message", "保存成功");
		} catch (Exception e) {
			logger.error("thridAppLogin:" + e.toString());
			jsonObject.put("code", "0");
			jsonObject.put("message", "保存失败，失败原因" + e.toString());
		} finally {
			return jsonObject.toString();
		}
	}


	/**
	 * MyDataManager板块 3.3.5 查询单个指标
	 */
	public String getSingleTarget(String target_id, String prov_id,
			String area_id, String report_id, String month, String type,
			String userId, String tokenId,String commonDimension) {
		JSONObject j = new JSONObject();
		if (!tempUtil.validUserToken(userId, tokenId)) {
			j.put("code", "102");
			j.put("message", UserTokenUtil.ERROR_MESSAGE);
			return j.toString();
		}
		String monthT = null;
		String accountName = null;
		String headName = null;
		String unit = null;
		try {
			//通用维度需求
			Map<String,String>commonDimensionMap=tempUtil.getCommonDimensionMap(report_id, commonDimension);
			//包装维度参数为特殊参数获取评论数据
			String specialAttr = null;
			if (commonDimensionMap == null) {
				specialAttr = "";
			} else {
				JSONObject specialAttrObj = new JSONObject();
				if (StringUtils.isBlank(commonDimension) || "null".equals(StringUtils.trim(commonDimension))) {
					specialAttrObj.put("commonDimension", "{\"data\":[{\"key\":\"" + commonDimensionMap.get("commonDimension") + "\"},{\"key\":\"" + commonDimensionMap.get("commonDimension") + "\"}]}");
				} else {
					JSONObject commonDimensionObj = JSONObject.fromObject(commonDimension);
					JSONArray commonDimensionData = commonDimensionObj.getJSONArray("data");
					if (commonDimensionData == null || commonDimensionData.size() == 0) {
						specialAttrObj.put("commonDimension", "{\"data\":[{\"key\":\"" + commonDimensionMap.get("commonDimension") + "\"},{\"key\":\"" + commonDimensionMap.get("commonDimension") + "\"}]}");
					} else {
						JSONObject commonDimensionData0 = commonDimensionData.getJSONObject(0);
						JSONObject commonDimensionData1 = null;
						try {
							commonDimensionData1 = commonDimensionData.getJSONObject(1);
						} catch (Exception e) {
							commonDimensionData1 = commonDimensionData0;
						}
						commonDimensionData.clear();
						commonDimensionData.add(commonDimensionData0);
						commonDimensionData.add(commonDimensionData1);
						commonDimensionObj.put("data", commonDimensionData);
						specialAttrObj.put("commonDimension", commonDimensionObj.toString());
					}
				}
				specialAttr = specialAttrObj.toString();
			}
			//update by gongsq  评论和关注
			String mainTargetId=target_id;
			List<CommunityComment> commentList = null;
			List<Map<String, Object>> isCollectList = communityDao.getTargetIsCollect(userId, report_id, type, mainTargetId, prov_id, area_id);
			String isCollect="0";
			if (CollectionUtils.isNotEmpty(isCollectList)) {
				isCollect="1";
			}
			//根据报表id查找指标的背景图片,0是没有,否则返回数据
			String targetImgPath="0";
			if (StringUtils.isNotBlank(report_id)){
				List<Map<String, Object>> targetImgPathList = scoreboardDao.getTargetImgPath(report_id);
				if (CollectionUtils.isNotEmpty(targetImgPathList)){
					targetImgPath= (String) targetImgPathList.get(0).get("IMG_PATH");
				}
			}
			// 获取省分个性化指标
			System.out.println("report_id==" + report_id);
			Map<String, Object> specialMap = dao.querSpecialReportConfig(
					report_id, ConstantMe.SPECIAL_METHOD_NAME_DYZB);
			if (null != specialMap) {
				headName = tempUtil.getHeadTitle(report_id);
				String areaName = (String) specialMap.get("AREA_NAME");
				JSONObject data = new JSONObject();
				tempUtil.addRecordAccessLog(userId,ConstantMe.SPECIAL_METHOD_NAME_DYZB, report_id, "", "","个性化报表配置", month);
				String url = String.valueOf(specialMap.get("URL"));
				String specialMethodName = String.valueOf(specialMap.get("SPECIAL_METHOD_NAME"));
				String specialProvId = String.valueOf(specialMap.get("PROV_ID"));
				if (StringUtils.isNotBlank(url) && StringUtils.isNotEmpty(url)&& StringUtils.isNotBlank(url)&& StringUtils.isNotEmpty(url)) {
					String responseStr = tempUtil.getSpecialReportInfo(url,month, report_id, target_id, userId, specialProvId,area_id, tokenId, specialMethodName,ConstantMe.SPECIAL_METHOD_NAME_DYZB);
					System.out.println("responseStr=======" + responseStr);
					data = JSONObject.fromObject(responseStr);
					if (data.containsKey("code")&& "1".equals(data.getString("code"))) {
						j = data.getJSONObject("data");
						j.put("month_id", data.getString("month_id"));
						j.put("head_title", headName);
						j.put("report_type", type);
						if (data.containsKey("area_name")) {
							j.put("area_name", data.getString("area_name"));
						} else {
							j.put("area_name", areaName);
						}
						j.put("report_level", "5");
						j.put("code", "1");
						month=data.getString("month_id");
						j.put("prov_id", prov_id);
						j.put("area_id", area_id);
						commentList = communityService.getTargetCommentList(report_id, type, mainTargetId, month, prov_id, area_id, specialAttr ,userId);
						if (CollectionUtils.isNotEmpty(commentList)) {
							j.put("commentList", JSON.toJSONString(commentList));
						}
						j.put("isCollect",isCollect);
						j.put("targetImgPath",targetImgPath);
						if (StringUtils.isNotBlank(specialAttr)){
							j.put("specialAttr",specialAttr);
						}
						if (data.containsKey("title_name")) {
							j.put("title_name", data.getString("title_name"));
						}
					} else {
						j = data;
					}
					System.out.println("个性化报表配置" + report_id
							+ "====返回的数据===========" + j.toString());
				}
				j.put("target_id", target_id);
				return j.toString();
			}
			// 判断是日报还是月报
			if (StringUtil.isEmpty(type)) {
				type = dao.queryTargetTypeById(target_id);
			}
			List<Map<String, Object>> confList = dao.getSingleTargetConfList(
					target_id, prov_id, area_id, report_id);
			// 获取报表对应的数据表
			String reportDataTable = "";
			if (StringUtil.isEmpty(month)) {
				if (ConstantMe.REPORT_TYPE_M.equals(type)) {
					month = dao.getLastestMonth();
					monthT = month.substring(0, 4) + "年"+ month.substring(4, 6) + "月";
				} else if (ConstantMe.REPORT_TYPE_D.equals(type)) {
					month = dao.getDayIdByReportId(report_id);
					monthT = month.substring(0, 4) + "年"+ month.substring(4, 6) + "月"+ month.substring(6, 8) + "日";
					reportDataTable = dao.getReportDataTable(report_id);
				} else if (ConstantMe.REPORT_TYPE_Q.equals(type)) {
					String quarterMonth = dao.getDayIdByReportId(report_id);
					monthT = quarterMonth.split("#")[1].substring(0, 4) + "年"
							+ quarterMonth.split("#")[1].substring(5, 6) + "季度";
					month = quarterMonth.split("#")[1].substring(0, 6);
				} else {
					List<Map<String, Object>> monthList = dao.getWeekAccountByReportId(report_id);
					if (monthList != null && monthList.size() > 0) {
						String monthStr = (String) monthList.get(0).get("VALUE");
						month = monthStr.split("#")[1];
						accountName = dao.getWeekAccountName(month);
						monthT = month;
					}
				}
			} else {
				if (ConstantMe.REPORT_TYPE_M.equals(type)) {
					monthT = month.substring(0, 4) + "年"+ month.substring(4, 6) + "月";
				} else if (ConstantMe.REPORT_TYPE_D.equals(type)) {
					monthT = month.substring(0, 4) + "年"+ month.substring(4, 6) + "月"+ month.substring(6, 8) + "日";
					reportDataTable = dao.getReportDataTable(report_id);
				} else if (ConstantMe.REPORT_TYPE_Q.equals(type)) {
					monthT = month.substring(0, 4) + "年"+ month.substring(5, 6) + "季度";
				}else {
					monthT = month;
					accountName = dao.getWeekAccountName(month);
				}
			}
            commentList = communityService.getTargetCommentList(report_id, type, mainTargetId, month, prov_id, area_id, specialAttr , userId);
			/** 增加显示区域名称 */
			String areaName = dao.getSingleAreaName(prov_id, area_id);
			List<Map<String, Object>> targetValueList = null;
			String targetsStr = "";
			if (confList != null && confList.size() > 0) {
				for (int i = 0; i < confList.size(); i++) {
					targetsStr += "'"+ (String) confList.get(i).get("TARGET_CODE") + "'"+ ",";
					if (StringUtils.isBlank(report_id)) {
						report_id = (String) confList.get(i).get("REPORT_ID");
					}
				}
			}
			targetsStr = targetsStr.equals("") ? "''" : targetsStr.substring(0,targetsStr.lastIndexOf(","));
			// 获取该模板展示的所有指标值
			if (ConstantMe.REPORT_TYPE_M.equals(type)) {
				targetValueList = dao.getAttrValuesBy(prov_id, area_id, month,targetsStr,commonDimensionMap);
			}else if (ConstantMe.REPORT_TYPE_Q.equals(type)) {
				targetValueList = dao.getAttrValuesByQuarter(prov_id, area_id, month,targetsStr,commonDimensionMap);
			}else if (ConstantMe.REPORT_TYPE_D.equals(type)) {
				if ("5000033".equals(report_id)) {
					targetValueList = dao.getInvestManageDayAttrValues(prov_id,area_id, month, targetsStr, "", "");
				} else {
					targetValueList = dao.getAttrValuesByDay(prov_id, area_id,month, targetsStr, reportDataTable);
				}
			} else {
				targetValueList = dao.getWeekAttrValuesBy(prov_id, area_id,month, targetsStr, "",commonDimensionMap);
			}
			if (confList == null || !(confList.size() > 0)) {
				j.put("code", "0");
				j.put("message", "找不到该指标的模板配置信息！");
				return j.toString();
			}
			month = month == null ? "" : month;
			j.put("month_id", monthT);
			String tempCode = (String) confList.get(0).get("TEMP_CODE");
			String targetName = (String) confList.get(0).get("MAIN_NAME");
			String targetUnit = (String) confList.get(0).get("MAIN_UNIT");
			String targetLevel = (String) confList.get(0).get("TARGET_LEVEL");
			String detail_show_flag = dao
					.getDetailShowFlagByReportId(report_id);
			String reportId = (String) confList.get(0).get("REPORT_ID");
			String targetExplain = dao.getTargetRemark(reportId, target_id,
					month);
			headName = tempUtil.getHeadTitle(reportId);
			if (reportId == null) {
				j.put("code", "0");
				j.put("message", "报表id为null!");
				return j.toString();
			}
			// 判断是否存在报告
			List<Map<String, Object>> reportList = null;
			String report_flag = "0";
			reportList = dao.getReportListFlag(target_id, prov_id, area_id,
					month);
			if (reportList.size() > 0) {
				report_flag = "1";
			}
			// 特殊处理模板39、40
			if (confList != null && confList.size() > 0) {
				if (confList.get(0).get("TEMP_ID").equals("39")) {
					MB_39_TempServiceImpl mb_39Data = new MB_39_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb39 = mb_39Data.getScoreBoard(report_id,target_id, prov_id, area_id, month, type,reportDataTable,commonDimension);
					jsonMb39.put("report_id", reportId);
					jsonMb39.put("area_name", areaName);
					jsonMb39.put("report_flag", report_flag);
					jsonMb39.put("head_title", headName);
					jsonMb39.put("month_id", monthT);
					jsonMb39.put("accountName", accountName);
					jsonMb39.put("detail_show_flag", detail_show_flag);
					jsonMb39.put("target_level", targetLevel);
					jsonMb39.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb39.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb39.put("isCollect",isCollect);
					jsonMb39.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb39.put("specialAttr",specialAttr);
                    }
					return jsonMb39.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("47")) {
					MB_47_TempServiceImpl mb_47Data = new MB_47_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb47 = mb_47Data.getScoreBoard(report_id,target_id, prov_id, area_id, month, type,reportDataTable, commonDimension);
					jsonMb47.put("report_id", reportId);
					jsonMb47.put("area_name", areaName);
				    jsonMb47.put("report_flag", report_flag);
					jsonMb47.put("head_title", headName);
					jsonMb47.put("month_id", monthT);
					jsonMb47.put("accountName", accountName);
					jsonMb47.put("detail_show_flag", detail_show_flag);
					jsonMb47.put("target_level", targetLevel);
					jsonMb47.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb47.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb47.put("isCollect",isCollect);
					jsonMb47.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb47.put("specialAttr",specialAttr);
                    }
					return jsonMb47.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("40")) {
					MB_40_TempServiceImpl mb_40Data = new MB_40_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb40 = mb_40Data.getScoreBoard(report_id,target_id, prov_id, area_id, month, type,reportDataTable,commonDimension);
					jsonMb40.put("report_id", reportId);
					jsonMb40.put("area_name", areaName);
					jsonMb40.put("report_flag", report_flag);
					jsonMb40.put("head_title", headName);
					jsonMb40.put("month_id", monthT);
					jsonMb40.put("accountName", accountName);
					jsonMb40.put("detail_show_flag", detail_show_flag);
					jsonMb40.put("target_level", targetLevel);
					jsonMb40.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb40.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb40.put("isCollect",isCollect);
					jsonMb40.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb40.put("specialAttr",specialAttr);
                    }
					return jsonMb40.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("42")) {
					MB_42_TempServiceImpl mb_42Data = new MB_42_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb42 = JSONObject.fromObject(mb_42Data.getScoreBoard(report_id, target_id, prov_id,area_id, month, type, reportDataTable));
					jsonMb42.put("report_id", reportId);
					jsonMb42.put("area_name", areaName);
					jsonMb42.put("report_flag", report_flag);
					jsonMb42.put("head_title", headName);
					jsonMb42.put("month_id", monthT);
					jsonMb42.put("accountName", accountName);
					jsonMb42.put("detail_show_flag", detail_show_flag);
					jsonMb42.put("target_level", targetLevel);
					jsonMb42.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb42.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb42.put("isCollect",isCollect);
					jsonMb42.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb42.put("specialAttr",specialAttr);
                    }
					return jsonMb42.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("45")) {
					MB_45_TempServiceImpl mb_45Data = new MB_45_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb42 = JSONObject.fromObject(mb_45Data.getScoreBoard(report_id, target_id, prov_id,area_id, month, type, reportDataTable));
					jsonMb42.put("report_id", reportId);
					jsonMb42.put("area_name", areaName);
					jsonMb42.put("report_flag", report_flag);
					jsonMb42.put("head_title", headName);
					jsonMb42.put("month_id", monthT);
					jsonMb42.put("accountName", accountName);
					jsonMb42.put("detail_show_flag", detail_show_flag);
					jsonMb42.put("target_level", targetLevel);
					jsonMb42.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb42.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb42.put("isCollect",isCollect);
					jsonMb42.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb42.put("specialAttr",specialAttr);
                    }
					return jsonMb42.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("43")&& ConstantMe.REPORT_TYPE_D.equals(type)) {
					MB_43_TempServiceImpl mb_43Data = new MB_43_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb43 = JSONObject.fromObject(mb_43Data.getScoreBoard(report_id, target_id, prov_id,area_id, month, reportDataTable));
					jsonMb43.put("report_id", reportId);
					jsonMb43.put("area_name", areaName);
					jsonMb43.put("report_flag", report_flag);
					jsonMb43.put("head_title", headName);
					jsonMb43.put("month_id", monthT);
					jsonMb43.put("accountName", accountName);
					jsonMb43.put("detail_show_flag", detail_show_flag);
					jsonMb43.put("target_level", targetLevel);
					jsonMb43.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb43.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb43.put("isCollect",isCollect);
					jsonMb43.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb43.put("specialAttr",specialAttr);
                    }
					return jsonMb43.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("46")) {
					MB_46_TempServiceImpl mb_46Data = new MB_46_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb40 = mb_46Data.getScoreBoard(report_id,target_id, prov_id, area_id, month, type,(String) confList.get(0).get("TEMP_ID"),(String) confList.get(0).get("TEMP_CODE"),(String) confList.get(0).get("TEMP_ORDER"),reportDataTable,commonDimension);
					jsonMb40.put("report_id", reportId);
					jsonMb40.put("area_name", areaName);
					jsonMb40.put("report_flag", report_flag);
					jsonMb40.put("head_title", headName);
					jsonMb40.put("month_id", monthT);
					jsonMb40.put("accountName", accountName);
					jsonMb40.put("detail_show_flag", detail_show_flag);
					jsonMb40.put("target_level", targetLevel);
					jsonMb40.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb40.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb40.put("isCollect",isCollect);
					jsonMb40.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb40.put("specialAttr",specialAttr);
                    }
					return jsonMb40.toString();
				} else if (confList.get(0).get("TEMP_ID").equals("44")) {
					MB_44_TempServiceImpl mb_44Data = new MB_44_TempServiceImpl(dao, tempUtil);
					JSONObject jsonMb40 = mb_44Data.getScoreBoard(report_id,target_id, prov_id, area_id, month, type,(String) confList.get(0).get("TEMP_ID"),(String) confList.get(0).get("TEMP_CODE"),(String) confList.get(0).get("TEMP_ORDER"),reportDataTable,commonDimension);
					jsonMb40.put("report_id", reportId);
					jsonMb40.put("area_name", areaName);
					jsonMb40.put("report_flag", report_flag);
					jsonMb40.put("head_title", headName);
					jsonMb40.put("month_id", monthT);
					jsonMb40.put("accountName", accountName);
					jsonMb40.put("detail_show_flag", detail_show_flag);
					jsonMb40.put("target_level", targetLevel);
					jsonMb40.put("target_id", target_id);
					if (CollectionUtils.isNotEmpty(commentList)) {
						jsonMb40.put("commentList", JSON.toJSONString(commentList));
					}
					jsonMb40.put("isCollect",isCollect);
					jsonMb40.put("targetImgPath",targetImgPath);
                    if (StringUtils.isNotBlank(specialAttr)){
                        jsonMb40.put("specialAttr",specialAttr);
                    }
					return jsonMb40.toString();
				}
			}
			unit = dao.getTargetUnitFromConfigInfo(target_id, report_id);
			j.put("report_flag", report_flag);
			j.put("temp_id", tempCode);
			j.put("area_name", areaName);
			j.put("report_id", reportId);
			if (CollectionUtils.isNotEmpty(commentList)) {
				j.put("commentList", JSON.toJSONString(commentList));
			}
			j.put("isCollect",isCollect);
			j.put("targetImgPath",targetImgPath);
            if (StringUtils.isNotBlank(specialAttr)){
                j.put("specialAttr",specialAttr);
            }
			j.put("detail_show_flag", detail_show_flag);
			j.put("target_level", targetLevel);
			j.put("target_name", targetName == null ? "" : targetName);
			// 20171214
			j.put("target_unit", unit == null ? "" : unit);
			j.put("target_explain", targetExplain == null ? "" : targetExplain);
			// add by lix 模板09需要加个是否把子指标传值到二级详情页面
			if ("mb_09".equals(tempCode)) {
				j.put("mb09_flag", "1");// 1：表示子指标。0：表示主指标
			}
			for (int i = 0; i < confList.size(); i++) {
				String regionCode = (String) confList.get(i).get("REGION_CODE");
				String targetId = String.valueOf(confList.get(i).get(
						"TARGET_ID"));
				String tarCod = (String) confList.get(i).get("TARGET_CODE");
				String targetN = (String) confList.get(i).get("TARGET_NAME");
				String attrC = (String) confList.get(i).get("ATTR_CODE");
				String attrN = (String) confList.get(i).get("ATTR_NAME");
				String showFlag = (String) confList.get(i).get("SHOW_FLAG") == null ? "1"
						: (String) confList.get(i).get("SHOW_FLAG");
				// 如果showFlag为3
				// 时取g_report_temp_config表中target_name字段,并把showFlag置为3
				if ("3".equals(showFlag)) {
					targetN = (String) confList.get(i)
							.get("TARGET_NAME_CONFIG");
				}
				if ("4".equals(showFlag)) {
					attrN = (String) confList.get(i).get("TARGET_NAME_CONFIG");
				}
				showFlag = showFlag.equals("4") ? "1" : showFlag;
				showFlag = showFlag.equals("3") ? "2" : showFlag;
				String unit_convert_ratio = (String) confList.get(i).get(
						"UNIT_CONVERT_RATIO");
				String confReportId = (String) confList.get(i).get("REPORT_ID");// 配置数据中的报表id
				if (!report_id.trim().equals(confReportId.trim())) {// 如果本次循环的配置数据report_id
																	// !=
																	// 开头确定的需要的report_id,退出本次循环
					continue;
				}
				for (int jj = 0; jj < targetValueList.size(); jj++) {
					String tarC = (String) targetValueList.get(jj).get("TARGET_CODE");
					if (tarC.equals(tarCod)) {
						Object tarV = targetValueList.get(jj).get(ConstantMe.YEAR_TOTAL) == null ? "--": targetValueList.get(jj).get(ConstantMe.YEAR_TOTAL);
						Object attrV = targetValueList.get(jj).get(attrC.toUpperCase()) == null ? "--": targetValueList.get(jj).get(attrC.toUpperCase());
						JSONObject job = new JSONObject();
						job.put("target_name", targetN);
						job.put("target_value", tempUtil.getCalcute((String) tarV,unit_convert_ratio, unit));
						job.put("target_id", targetId);
						job.put("show_flag", showFlag);
						job.put("attr_name", attrN);
						// 拿出config_2表中的单位进行判断如果config_2为空则取target_info表中单位
						String target_unit = confList.get(i).get("MAIN_UNIT") == null ? "--": confList.get(i).get("MAIN_UNIT").toString();
						if ("--".equals(target_unit)) {
							target_unit = confList.get(i).get("UNIT") == null ? "--": confList.get(i).get("UNIT").toString();
						}
						if (attrV == null) {
							attrV = tempUtil.getCalcute((String) attrV,unit_convert_ratio, unit);
						}
						// 所有占比环比等这类的比率直接取数据库里的target_unti字段 不在代码里写死
						if ((attrC.indexOf("ratio") != -1|| target_unit.indexOf("%") != -1 || target_unit.indexOf("pp") != -1)&& attrV.toString().indexOf("%") == -1&& attrV.toString().toUpperCase().indexOf("pp".toUpperCase()) == -1) {
							if (!"--".equals(attrV)) {
								attrV = attrV + target_unit;
							}
							// 不针对环比之类的比率进行数据处理
						} else {
							// 不针对环比之类的比率进行数据处理
							attrV = tempUtil.getCalcute((String) attrV,unit_convert_ratio, unit);
						}
						job.put("attr_value", attrV);
						j.put("attr_" + regionCode, job.toString());
					}
				}
				if (j.get("attr_" + regionCode) == null) {
					JSONObject job = new JSONObject();
					job.put("target_name", targetN);
					job.put("show_flag", showFlag);
					job.put("target_value", "--");
					job.put("attr_name", attrN);
					job.put("attr_value", "--");
					j.put("attr_" + regionCode, job.toString());
				}
			}
			j.put("prov_id", prov_id);
			j.put("area_id", area_id);
			j.put("report_type", type);
		} catch (Exception e) {
			logger.info("错误信息为：" + ExceptionUtil.getStackTrace(e));
			j.put("code", "0");
			j.put("message", "单一指标展示异常！");
			tempUtil.addErrRecordAccessLog("", "getSingleTarget", report_id,target_id, "","模糊搜索-查看指标异常：" + String.valueOf(e.getMessage()), "");
			return j.toString();
		}
		j.put("head_title", headName);
		j.put("accountName", accountName);
		j.put("code", "1");
		j.put("target_id", target_id);
		System.out.println("getsingletarget报表配置" + report_id
				+ "====返回的数据===========" + j.toString());
		return j.toString();
	}
}
11111111111111