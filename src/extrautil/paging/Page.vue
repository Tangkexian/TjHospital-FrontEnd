package com.javaee.onlinehosbackend.extrautil.paging;

import java.util.List;

/**
 * 分页对象
 * 
 * @Author Sugar
 * @Version 2017年3月30日 下午8:08:58
 */
public class Page implements IPage {
	private static final long serialVersionUID = 1L;

	private long totalCount;// 总记录数
	private int pageCount;// 总页数
	private int pageSize;// 每页记录数
	private int pageNo;// 当前页页码，从1开始
	private int prevPageNo;// 上一页页码，为0时无上一页
	private int nextPageNo;// 下一页页码，为0时无下一页
	private List<?> list;// 本页数据集
	private String url = "javascript:page(#page#)";//分页跳转连接
	private String orderBy;
	private Class clazz;
	private long maxSize;//最大显示记录数
	private int endPageNo;//最后一页页码

	public Page() {
		pageNo = 1;
		pageSize = 20;
	}

	public Page(int pageSize, int pageNo) {
		this.pageSize = pageSize;
		this.pageNo = (pageNo < 1 ? 1 : pageNo);
	}
	
	public Page(int pageSize, int pageNo, String url) {
		this.pageSize = pageSize;
		this.pageNo = (pageNo < 1 ? 1 : pageNo);
		this.url = url;
	}

	/**
	 * @param totalCount
	 *            数据总记录数
	 * @param pageSize
	 *            每页显示多少条数据
	 * @param pageNo
	 *            当前第几页，从1开始
	 * @param list 当前数据列表
	 */
	public <T> Page(long totalCount, int pageSize, int pageNo, List<T> list) {
		this.totalCount = totalCount;
		this.pageSize = pageSize;
		this.pageNo = (pageNo < 1 ? 1 : pageNo);
		this.list = list;
		init();
	}

	/**
	 *
	 * @param pageSize 每页显示多少条数据
	 * @param pageNo 当前第几页，从1开始
	 * @param dataList dataList 所有数据列表
	 */
	public <T> Page(int pageSize, int pageNo, List<T> dataList) {
		if (dataList == null) {
			return;
		}
		this.totalCount = dataList.size();
		this.pageSize = pageSize;
		this.pageNo = (pageNo < 1 ? 1 : pageNo);
		int start = pageSize * (this.pageNo - 1);
		int end = start + pageSize;
		if (end <= totalCount) {
			this.list = dataList.subList(start, end);
		} else if (start < totalCount) {
			this.list = dataList.subList(start, (int) totalCount);
		}
		init();
	}

	public long getTotalCount() {
		return totalCount;
	}

	public void setTotalCount(long totalCount) {
		this.totalCount = totalCount;
		init();
	}

	public int getPageCount() {
		return pageCount;
	}

	public int getPageSize() {
		return pageSize;
	}

	public void setPageSize(int pageSize) {
		if (this.pageSize != pageSize) {
			this.pageSize = pageSize;
			init();
		}
	}

	public int getPageNo() {
		return pageNo;
	}

	public void setPageNo(int pageNo) {
		this.pageNo = pageNo;
	}

	/**
	 * 跳转地址，如：/xxx/server/?page=#page#&param1=xxx，默认为javascript:page(#page#)
	 * @param url
	 */
	public void setUrl(String url) {
		this.url = url;
	}

	/**
	 * @return the orderBy
	 */
	public String getOrderBy() {
		return orderBy;
	}

	/**
	 * @param orderBy the orderBy to set
	 */
	public void setOrderBy(String orderBy) {
		this.orderBy = orderBy;
	}
	
	@SuppressWarnings("unchecked")
	public <T> List<T> getList() {
		return (List<T>) list;
	}

	public <T> void setList(List<T> list) {
		this.list = list;
	}

	public int getPrevPageNo() {
		return prevPageNo;
	}

	public void setPrevPageNo(int prevPageNo) {
		this.prevPageNo = prevPageNo;
	}

	public int getNextPageNo() {
		return nextPageNo;
	}

	public void setNextPageNo(int nextPageNo) {
		this.nextPageNo = nextPageNo;
	}

	public String toString() {
		int curPageNo = getPageNo();
		StringBuffer nav = new StringBuffer("<ul class=\"pagination\">");
		if (curPageNo <= 1) {
			nav.append("<li class=\"previous disabled\"><a href=\"javascript:;\"><i class=\"glyphicon glyphicon-menu-left\"></i></a></li>");
		} else {
			nav.append("<li class=\"previous\" data=\"" + (curPageNo - 1) + "\"><a href=\"" + url.replace("#page#", String.valueOf(curPageNo - 1)) + "\">上一页</a></li>");
		}
		if (endPageNo <= 7) {
			for (int i = 1; i <= endPageNo; i++) {
				if (i == curPageNo) {
					nav.append("<li class=\"active\"><a href=\"javascript:;\">");
				} else {
					nav.append("<li data=\"" + i + "\"><a href=\"" + url.replace("#page#", String.valueOf(i)) + "\">");
				}
				nav.append(i).append("</a></li>");
			}
		} else {
			if (curPageNo <= 4) {
				for (int i = 1; i <= 5; i++) {
					if (i == curPageNo) {
						nav.append("<li class=\"active\"><a href=\"javascript:;\">");
					} else {
						nav.append("<li data=\"" + i + "\"><a href=\"" + url.replace("#page#", String.valueOf(i)) + "\">");
					}
					nav.append(i).append("</a></li>");
				}
				nav.append("<li class=\"disabled\"><a href=\"javascript:;\">…</a></li>");
				nav.append("<li data=\"" + endPageNo + "\"><a href=\"" + url.replace("#page#", String.valueOf(endPageNo)) + "\">" + endPageNo + "</a></li>");
			} else if (curPageNo + 5 >= endPageNo) {
				nav.append("<li data=\"1\"><a href=\"" + url.replace("#page#", String.valueOf(1)) + "\">1</a></li>");
				nav.append("<li class=\"disabled\"><a href=\"javascript:;\">…</a></li>");
				for (int i = endPageNo - 5; i <= endPageNo; i++) {
					if (i == curPageNo) {
						nav.append("<li class=\"active\"><a href=\"javascript:;\">");
					} else {
						nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(i)) + "\">");
					}
					nav.append(i).append("</a></li>");
				}
			} else {
				nav.append("<li data=\"1\"><a href=\"" + url.replace("#page#", String.valueOf(1)) + "\">1</a></li>");
				nav.append("<li class=\"disabled\"><a href=\"javascript:;\">…</a></li>");
				nav.append("<li data=\"" + (curPageNo - 1) + "\"><a href=\"" + url.replace("#page#", String.valueOf(curPageNo - 1)) + "\">").append(curPageNo - 1)
						.append("</a></li>");
				nav.append("<li class=\"active\"><a href=\"javascript:;\">").append(curPageNo).append("</a></li>");
				nav.append("<li data=\"" + (curPageNo + 1) + "\"><a href=\"" + url.replace("#page#", String.valueOf(curPageNo + 1)) + "\">").append(curPageNo + 1)
						.append("</a></li>");
				nav.append("<li class=\"disabled\"><a href=\"javascript:;\">…</a></li>");
				nav.append("<li data=\"" + endPageNo + "\"><a href=\"" + url.replace("#page#", String.valueOf(endPageNo)) + "\">" + endPageNo + "</a></li>");
			}
		}
		if (pageCount > endPageNo) {
			nav.append("<li class=\"disabled\"><a href=\"javascript:;\" title=\"查询结果数超过最大限制"+maxSize+"条，请使用更精确的查询条件\">…</a></li>");
		}
		if (curPageNo >= endPageNo) {
			nav.append("<li class=\"next disabled\"><a href=\"javascript:;\"><i class=\"glyphicon glyphicon-menu-right\"></i></a></li>");
		} else {
			nav.append("<li class=\"next\" data=\"" + (curPageNo + 1) + "\"><a href=\"" + url.replace("#page#", String.valueOf(curPageNo + 1)) + "\"><i class" +
					"=\"glyphicon glyphicon-menu-right\"></i></a></li>");
		}
		nav.append("</ul>");
		return nav.toString();
	}
	
	public String getMessage() {
		if (this.totalCount == 0) {
			return "<span>共 0 条</span>";
		}
		StringBuilder str = new StringBuilder();
		if (url.contains("/")) {
			str.append("<span class=\"page-skip\">到第<input id=\"skipToPage\" type=\"text\" min=\"1\" value=\"" + pageNo + "\" class=\"form-control\">页<button" +
					" " +
					"type=\"button\" " +
					"class=\"btn btn-default\" onclick=\"javascript:location.href='" + url.replace("#page#", "'+skipToPage.value+'") +
					"';\">确定</button></span>");
		} else {
			str.append("<span class=\"page-skip\">到第<input id=\"skipToPage\" type=\"text\" min=\"1\" value=\"" + pageNo + "\" class=\"form-control\">页<button" +
					" " +
					"type=\"button\" " +
					"class=\"btn btn-default\" onclick=\"" + url.replace("#page#", "skipToPage.value") +
					"\">确定</button></span>");
		}
		str.append("<span class=\"page-count\">共 "+this.totalCount+" 条</span>");
		str.append("<span class=\"page-size\"><select id=\"setPageSize\" class=\"form-control\">");
		for (int i = 10; i <= 100; i+=10) {
			if (pageSize == i) {
				str.append("<option value=\""+i+"\" selected>"+i+" 条/页</option>");
			} else {
				str.append("<option value=\""+i+"\">"+i+" 条/页</option>");
			}
		}
		str.append("</select></span>");
		return str.toString();
	}

	public String getSimplePagination(String url) {
		return getSimplePagination(url, 7);
	}

	/**
	 * 简单的分页导航
	 * 
	 * @param url
	 * @param num
	 * @return
	 */
	public String getSimplePagination(String url, int num) {
		if (url == null) {
			url = "";
		}
		int curPageNo = getPageNo();
		StringBuffer nav = new StringBuffer("<ul class=\"pagination\">");
		if (endPageNo <= num) {
			for (int i = 1; i <= endPageNo; i++) {
				if (i == curPageNo) {
					nav.append("<li class=\"active\"><a href=\"javascript:;\">");
				} else {
					nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(i)) + "\">");
				}
				nav.append(i).append("</a></li>");
			}
		} else {
			if (curPageNo + num - 1 >= endPageNo) {
				nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(1)) + "\">&lt;&lt;</a></li>");
				nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(prevPageNo)) + "\">&lt;</a></li>");
				for (int i = endPageNo - num - 1; i <= endPageNo; i++) {
					if (i == curPageNo) {
						nav.append("<li class=\"active\"><a href=\"javascript:;\">");
					} else {
						nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(i)) + "\">");
					}
					nav.append(i).append("</a></li>");
				}
			} else {
				if (curPageNo > 1) {
					nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(1)) + "\">&lt;&lt;</a></li>");
					nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(prevPageNo)) + "\">&lt;</a></li>");
				}
				for (int i = curPageNo; i < curPageNo + num; i++) {
					if (i == curPageNo) {
						nav.append("<li class=\"active\"><a href=\"javascript:;\">");
					} else {
						nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(i)) + "\">");
					}
					nav.append(i).append("</a></li>");
				}
				nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(nextPageNo)) + "\">&gt;</a></li>");
				nav.append("<li><a href=\"" + url.replace("#page#", String.valueOf(endPageNo)) + "\">&gt;&gt;</a></li>");
			}
		}
		nav.append("</ul>");
		return nav.toString();
	}

	public void init() {
		if (totalCount % this.pageSize == 0) {
			this.pageCount = (int) (totalCount / this.pageSize);
		} else {
			this.pageCount = (int) (totalCount / this.pageSize + 1);
		}
		this.endPageNo = this.pageCount;
		if (this.maxSize > 0 && this.maxSize < this.totalCount) {//如果限制了最大记录数，最大页数只能以最大记录数计算
			this.endPageNo = (int)(this.maxSize / this.pageSize);
		}
		if (this.pageNo < 1) {
			this.pageNo = 1;
		} else if (this.endPageNo > 0 && this.pageNo > this.endPageNo) {
			this.pageNo = this.endPageNo;
		}
		this.prevPageNo = this.pageNo - 1;
		if (this.pageNo == this.endPageNo) {
			this.nextPageNo = 0;
		} else {
			this.nextPageNo = this.pageNo + 1;
		}
	}

	public <T> Class<T> getClazz() {
		return clazz;
	}

	public void setClazz(Class clazz) {
		this.clazz = clazz;
	}

	public void setMaxSize(long maxSize) {
		this.maxSize = maxSize;
	}

	public void reset() {
		this.pageNo = 1;
		this.prevPageNo = 0;
		this.nextPageNo = 0;
		this.pageCount = 0;
		this.list = null;
		this.totalCount = 0;
	}
}
