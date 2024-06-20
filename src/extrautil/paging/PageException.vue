package com.javaee.onlinehosbackend.extrautil.paging;

/**
 * 分页异常
 * @Auther Sugar
 * @Version	2017-07-31
 */
public class PageException extends RuntimeException {
	public PageException() {
		super();
	}

	public PageException(String message) {
		super(message);
	}

	public PageException(String message, Throwable cause) {
		super(message, cause);
	}

	public PageException(Throwable cause) {
		super(cause);
	}
}
