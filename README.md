# 4-2-Assignment-Game-Player-Management-Application
GameAuthenticator.java

package com.gamingroom.gameauth.auth;

import io.dropwizard.auth.AuthenticationException;
import io.dropwizard.auth.Authenticator;
import io.dropwizard.auth.basic.BasicCredentials;

import java.util.Map;
import java.util.Optional;
import java.util.Set;

import com.google.common.collect.ImmutableMap;
import com.google.common.collect.ImmutableSet;

public class GameAuthenticator implements Authenticator<BasicCredentials, GameUser> {

    private static final Map<String, Set<String>> VALID_USERS = ImmutableMap.of(
            "guest", ImmutableSet.of(),
            "user", ImmutableSet.of("USER"),
            "admin", ImmutableSet.of("ADMIN", "USER")
    );

    @Override
    public Optional<GameUser> authenticate(BasicCredentials credentials) throws AuthenticationException {
        if (VALID_USERS.containsKey(credentials.getUsername()) && "password".equals(credentials.getPassword())) {
            return Optional.of(new GameUser(credentials.getUsername(), VALID_USERS.get(credentials.getUsername())));
        }
        return Optional.empty();
    }
}

GameAuthorizer.java

package com.gamingroom.gameauth.auth;

import io.dropwizard.auth.Authorizer;

public class GameAuthorizer implements Authorizer<GameUser> {

    @Override
    public boolean authorize(GameUser user, String role) {
        return user.getRoles() != null && user.getRoles().contains(role);
    }
}

GameUser.java

package com.gamingroom.gameauth.auth;

import java.security.Principal;
import java.util.Set;

public class GameUser implements Principal {

    private String name = "";
    private final Set<String> roles;

    public GameUser(String name) {
        this.name = name;
        this.roles = null;
    }

    public GameUser(String name, Set<String> roles) {
        this.name = name;
        this.roles = roles;
    }

    public String getName() {
        return name;
    }

    public Set<String> getRoles() {
        return roles;
    }
}

GameUserRESTController.java

package com.gamingroom.gameauth.controller;

import io.dropwizard.auth.Auth;

import javax.annotation.security.PermitAll;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import com.gamingroom.gameauth.auth.GameUser;
import com.gamingroom.gameauth.dao.GameUserDB;

@Path("/gameusers")
@Produces(MediaType.APPLICATION_JSON)
public class GameUserRESTController {

    @PermitAll
    @GET
    public Response getGameUsers(@Auth GameUser user) {
        return Response.ok(GameUserDB.getGameUsers()).build();
    }
}
