# $NetBSD$
# -*- mode: makefile; tab-width: 8; indent-tabs-mode: 1 -*-
# vim: ts=8 sw=8 ft=make noet

DISTNAME=		redundis-${VERSION}
VERSION?=		0.0.0
# PKGREVISION?=		1
CATEGORIES=		pagoda
MASTER_SITES=		github.com/mu-box/redundis

MAINTAINER=		pkgsrc-users@NetBSD.org
HOMEPAGE=		github.com/mu-box/redundis
COMMENT=		Redundis is a redis redundancy/auto-failover solution
LICENSE=       		MIT

USE_TOOLS+=		gmake

WRKSRC=			${WRKDIR}/redundis

GIT_REPOSITORIES=	redundis
GIT_REPO.redundis=	git@github.com:mu-box/redundis.git
# GIT_TAG=		${VERSION}
GIT_BRANCH.redundis=	master

.include "../../base/git/git-package.mk"
.include "../../base/luvit/buildlink3.mk"
.include "../../mk/bsd.pkg.mk"

