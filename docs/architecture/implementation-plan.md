# Sprint 2 Implementation Plan

| Property | Value |
|----------|-------|
| Project | Principal Consultant Playbook |
| Sprint | Sprint 2 |
| Artifact | S2-A003 |
| Version | 1.0 |
| Status | Draft |
| Author | Greg Mitchell |
| Last Updated | 2026-08-04 |

---

# Objective

Transform the baseline F5 lab into a production-style application delivery environment by introducing multiple web servers and validating enterprise load-balancing behavior.

---

# Scope

This sprint will implement:

- Add Ubuntu-Web02
- Expand WEB_POOL to multiple members
- Validate load balancing
- Configure health monitoring
- Test automatic failover
- Update documentation

---

# Out of Scope

- SSL Bridging
- iRules
- ACI Integration
- Automation
- Database tier

---

# Success Criteria

The sprint is considered successful when:

- [ ] Ubuntu-Web02 is operational.
- [ ] WEB_POOL contains multiple healthy members.
- [ ] Client traffic is distributed across both servers.
- [ ] Health monitors remove failed servers automatically.
- [ ] Documentation is updated.
- [ ] Changes are committed to Git.

---

# Risks

| Risk | Mitigation |
|------|------------|
| Incorrect IP configuration | Verify addressing before implementation |
| Apache service failure | Validate service before adding to pool |
| Incorrect pool configuration | Test with one member before adding the second |
| Documentation drift | Update documentation immediately after changes |

---

# Rollback Plan

If implementation fails:

1. Remove Ubuntu-Web02 from WEB_POOL.
2. Restore WEB_POOL to a single member.
3. Verify client connectivity.
4. Review logs before retrying.

---

# Validation Plan

- Verify BIG-IP pool status.
- Verify VIP availability.
- Confirm HTTP responses from both servers.
- Simulate a server failure.
- Verify uninterrupted client access.

---

# Deliverables

- Updated physical topology
- Updated IP addressing plan
- Updated device inventory
- Updated pool documentation
- Git commit